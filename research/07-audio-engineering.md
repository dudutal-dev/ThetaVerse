# Audio Engineering Research Report: In-Browser Sound Generation for a ThetaHealing Frequencies Module

**Target**: Single-file HTML PWA, Hebrew RTL, mobile-first, all audio synthesized via Web Audio API (no audio files, or tiny base64 only).

---

## 1. Web Audio API Generation Techniques

### 1.0 Core setup — one shared graph

Build **one** `AudioContext` (created/resumed only inside a user gesture — see §2), one master chain, and hang all voices off it:

```js
let ctx = null, master = null, limiter = null;

function initAudio() {                 // call ONLY from a tap/click handler
  if (ctx) return ctx.resume();
  ctx = new (window.AudioContext || window.webkitAudioContext)({
    latencyHint: 'playback',           // bigger buffers => fewer glitches + less battery
    sampleRate: 48000                  // request; iOS may ignore — never assume, read ctx.sampleRate
  });
  limiter = ctx.createDynamicsCompressor();
  limiter.threshold.value = -12; limiter.knee.value = 30;
  limiter.ratio.value = 12; limiter.attack.value = 0.003; limiter.release.value = 0.25;
  master = ctx.createGain();
  master.gain.value = 0.8;
  master.connect(limiter).connect(ctx.destination);
  return ctx.resume();
}
```

The `DynamicsCompressorNode` at the end acts as a **safety limiter**: whatever the user stacks (binaural + noise bed + bell strike), the sum never clips or spikes painfully in earbuds. This is standard practice for generative-audio apps.

### 1.1 Pure tones + click-free envelopes

Never start/stop an oscillator at nonzero gain — you get a click. Always wrap each voice in its own `GainNode` and ramp:

```js
function playTone(freq, { attack = 0.05, release = 0.3 } = {}) {
  const osc = ctx.createOscillator();      // default 'sine'
  const g = ctx.createGain();
  osc.frequency.value = freq;
  g.gain.value = 0.0001;                   // exponential ramps can't start at 0
  osc.connect(g).connect(master);
  const t = ctx.currentTime;
  osc.start(t);
  g.gain.setValueAtTime(0.0001, t);
  g.gain.exponentialRampToValueAtTime(0.25, t + attack);   // exponential = perceptually smooth
  return {
    stop() {
      const t2 = ctx.currentTime;
      g.gain.cancelScheduledValues(t2);
      g.gain.setValueAtTime(g.gain.value, t2);             // anchor before ramp (required)
      g.gain.exponentialRampToValueAtTime(0.0001, t2 + release);
      osc.stop(t2 + release + 0.05);                       // frees resources; osc nodes are one-shot
    }
  };
}
```

Key rules:
- `exponentialRampToValueAtTime` cannot pass through 0 — use 0.0001 as floor, or use `setTargetAtTime(0, t, timeConstant)` which decays asymptotically (great for natural releases; audible done after ~5× timeConstant).
- Always `setValueAtTime(currentValue, now)` before a new ramp, or the ramp starts from the last *scheduled* point, not from where it audibly is.
- Oscillator/Buffer source nodes are **single-use**: create new ones per note; they garbage-collect after `stop()`.

### 1.2 Binaural beats

Two sine oscillators, hard-panned: left ear gets carrier `f`, right ear gets `f + Δ`; the brain perceives a Δ-Hz beat. Δ = target brainwave band (theta = 4–8 Hz), carrier typically 100–400 Hz (200–250 Hz is the comfortable sweet spot; entrainment literature suggests carriers < ~1000 Hz work best).

```js
function createBinaural(carrier = 200, beat = 6, level = 0.25) {
  const oscL = ctx.createOscillator(), oscR = ctx.createOscillator();
  oscL.frequency.value = carrier;
  oscR.frequency.value = carrier + beat;

  // StereoPannerNode approach (simplest, widely supported incl. iOS 14.5+):
  const panL = new StereoPannerNode(ctx, { pan: -1 });
  const panR = new StereoPannerNode(ctx, { pan: 1 });
  const g = ctx.createGain(); g.gain.value = 0.0001;
  oscL.connect(panL).connect(g);
  oscR.connect(panR).connect(g);
  g.connect(master);
  oscL.start(); oscR.start();
  g.gain.exponentialRampToValueAtTime(level, ctx.currentTime + 3); // slow 3s fade-in is the norm
  return { oscL, oscR, gain: g,
    setBeat(b) { oscR.frequency.setTargetAtTime(carrier + b, ctx.currentTime, 0.5); } };
}
```

`ChannelMergerNode` alternative (guaranteed true channel isolation, works even on ancient WebKit without StereoPanner):

```js
const merger = ctx.createChannelMerger(2);
oscL.connect(gL).connect(merger, 0, 0);   // input 0 -> left channel
oscR.connect(gR).connect(merger, 0, 1);   // input 0 -> right channel
merger.connect(masterGain);
```

**Headphone detection: not reliably possible.** There is no web API that tells you "headphones are connected" (`navigator.mediaDevices.enumerateDevices()` lists *input* devices, needs mic permission, and is unreliable for outputs on iOS; `setSinkId` is Chrome-only and doesn't detect). Also note: **iPhone speakers can be effectively mono / channels acoustically mixed**, and Bluetooth speakers downmix — binaural beats simply don't work there. **Standard solution is UX, not detection**: show a headphones icon + one-line notice on any binaural preset ("🎧 מומלץ עם אוזניות — הצליל הבינאורלי עובד רק כך"), and optionally offer "אין לי אוזניות" → auto-switch the preset to isochronic mode (same Δ frequency), which works on speakers. This carrier-swap fallback is exactly what several commercial apps do.

### 1.3 Isochronic tones (work on speakers — no headphones needed)

Amplitude-modulate a carrier at the target rate. Cleanest graph: **LFO oscillator → gain.gain** (AudioParam modulation, runs on the audio thread, zero drift):

```js
function createIsochronic(carrier = 200, pulseHz = 2.5, level = 0.3, shape = 'smooth') {
  const osc = ctx.createOscillator(); osc.frequency.value = carrier;
  const am = ctx.createGain(); am.gain.value = 0;        // depth container
  const lfo = ctx.createOscillator(); lfo.frequency.value = pulseHz;
  const lfoDepth = ctx.createGain(); lfoDepth.gain.value = 0.5;
  const dc = ctx.createConstantSource(); dc.offset.value = 0.5;  // shift LFO from [-1,1] to [0,1]
  lfo.connect(lfoDepth).connect(am.gain);
  dc.connect(am.gain);
  const out = ctx.createGain(); out.gain.value = level;
  osc.connect(am).connect(out).connect(master);
  if (shape === 'square') lfo.type = 'square';           // classic hard isochronic pulse
  // 'smooth' (sine LFO) = gentler, less fatiguing for long meditation sessions
  osc.start(); lfo.start(); dc.start();
  return { lfo, out };
}
```

Pulse shaping tradeoff: **square** LFO gives the sharp on/off pulses that entrainment purists want (stronger auditory evoked response) but is harsher over 20+ minutes and produces clicky sidebands; a raw square also clicks at edges — better is a **trapezoid** via `setValueCurveAtTime`, or simply a **sine LFO** ("smooth mode") which most consumer apps use by default. Offer both; default smooth. Keep modulation depth ~80–100% (full on/off) for square, and note that for very low pulse rates (delta, <3 Hz) isochronic pulses become individually audible thumps — pick a soft shape there.

### 1.4 Solfeggio tones with pleasant timbre

The 9 solfeggio frequencies: **174, 285, 396, 417, 528, 639, 741, 852, 963 Hz**. A naked sine at 528 Hz is thin and clinical. Two cheap fixes:

**(a) Layer soft harmonics** (additive, one extra osc per harmonic):

```js
function solfeggioVoice(f, level = 0.25) {
  const out = ctx.createGain(); out.gain.value = 0.0001;
  const partials = [ [1, 1.0], [2, 0.25], [3, 0.12] ];   // fundamental + 2nd/3rd at low gain
  const oscs = partials.map(([mult, amp]) => {
    const o = ctx.createOscillator(), g = ctx.createGain();
    o.frequency.value = f * mult; g.gain.value = amp;
    o.detune.value = (Math.random() - 0.5) * 4;          // ±2 cents: subtle chorus, less sterile
    o.connect(g).connect(out); o.start(); return o;
  });
  out.connect(master);
  out.gain.exponentialRampToValueAtTime(level, ctx.currentTime + 2);
  return { oscs, out };
}
```

**(b) Or one oscillator with a custom `PeriodicWave`** (identical spectrum, single node):

```js
const real = new Float32Array([0, 1, 0.25, 0.12]);   // harmonic amplitudes (cos terms)
const imag = new Float32Array(real.length);           // zeros
osc.setPeriodicWave(ctx.createPeriodicWave(real, imag, { disableNormalization: false }));
```

Extra polish: add a very slow tremolo (0.1–0.25 Hz sine LFO, ±10% gain) so long tones "breathe" instead of sounding like a test signal — this is what makes generated drones feel like meditation apps rather than hearing tests. A second copy detuned ±3 cents panned slightly L/R gives width without being binaural.

### 1.5 Meditation bells / singing bowls (synthesized — no sample needed)

Bells are defined by **inharmonic partials** (non-integer ratios) with **independent exponential decays** (higher partials die faster). Additive synthesis is simple and sounds convincingly like a strike bowl:

```js
// Tibetan-bowl-ish strike. baseFreq ~ 220-520 works well; when=absolute AudioContext time.
function strikeBowl(when = ctx.currentTime, baseFreq = 440, level = 0.5) {
  // ratio, relative amp, decay seconds — inharmonic set measured from real bowls
  const partials = [
    [1.0,   1.00, 6.0],
    [2.756, 0.60, 4.5],   // classic bell-mode ratios (~2.76, 5.4, 8.9 for bowls)
    [5.404, 0.35, 3.0],
    [8.933, 0.20, 2.0],
    [13.34, 0.10, 1.2],
    [1.003, 0.60, 6.0],   // near-unison pair => the characteristic slow "wah" beating
  ];
  const out = ctx.createGain(); out.gain.value = level;
  out.connect(master);
  for (const [ratio, amp, decay] of partials) {
    const o = ctx.createOscillator(), g = ctx.createGain();
    o.frequency.value = baseFreq * ratio;
    g.gain.setValueAtTime(0.0001, when);
    g.gain.linearRampToValueAtTime(amp, when + 0.005 + Math.random() * 0.01); // fast, slightly scattered attack
    g.gain.setTargetAtTime(0.0001, when + 0.02, decay / 5);                    // exponential ring-out
    o.connect(g).connect(out);
    o.start(when); o.stop(when + decay + 1);
  }
}
```

Notes:
- The **detuned near-unison pair** (ratios 1.0 and 1.003) is the secret: it produces the slow amplitude beating that says "singing bowl" to the ear.
- FM alternative (2 nodes instead of 12): modulator osc at `f * 1.4` (non-integer ratio ⇒ inharmonic spectrum) connected to `carrier.frequency` through a gain whose value decays from ~`f*3` to 0 over ~1 s — a classic DX7-style bell. Cheaper but harder to tune tastefully; the additive version above is recommended.
- Adding a 5 ms burst of filtered noise at the attack ("chiff") increases realism, but is optional.
- **Base64 sample alternative**: a real bowl strike at 22.05 kHz mono, ~3–4 s, Opus/MP3 ~24 kbps ≈ 10–15 KB ⇒ ~14–20 KB base64. Decode once via `ctx.decodeAudioData(...)` into an `AudioBuffer`, replay with `AudioBufferSourceNode` (vary `playbackRate` 0.9–1.1 for pitch variety). Legit within the "tiny base64 only" budget if the synthetic bell doesn't satisfy — but try the synth first; it's genuinely good and costs 0 bytes.

### 1.6 Ambient beds: noise + filters

Generate a **looped noise `AudioBuffer`** once (simplest, no AudioWorklet file needed — important for a *single-file* app, since `audioWorklet.addModule()` wants a URL; you *can* inline a worklet via `Blob` URL, but a looped buffer is simpler and battery-cheaper). Use ≥5 s (ideally 10 s or a prime-ish length) so the loop isn't perceptible:

```js
function makeNoiseBuffer(type = 'pink', seconds = 10) {
  const sr = ctx.sampleRate, buf = ctx.createBuffer(2, sr * seconds, sr);
  for (let ch = 0; ch < 2; ch++) {
    const d = buf.getChannelData(ch);
    if (type === 'white') { for (let i = 0; i < d.length; i++) d[i] = Math.random() * 2 - 1; }
    else if (type === 'pink') {            // Paul Kellet's economy filter — the standard JS recipe
      let b0=0,b1=0,b2=0,b3=0,b4=0,b5=0,b6=0;
      for (let i = 0; i < d.length; i++) {
        const w = Math.random() * 2 - 1;
        b0 = 0.99886*b0 + w*0.0555179; b1 = 0.99332*b1 + w*0.0750759;
        b2 = 0.96900*b2 + w*0.1538520; b3 = 0.86650*b3 + w*0.3104856;
        b4 = 0.55000*b4 + w*0.5329522; b5 = -0.7616*b5 - w*0.0168980;
        d[i] = (b0+b1+b2+b3+b4+b5+b6 + w*0.5362) * 0.11;
        b6 = w * 0.115926;
      }
    } else {                               // brown: integrated white noise, deepest/softest
      let last = 0;
      for (let i = 0; i < d.length; i++) {
        const w = Math.random() * 2 - 1;
        d[i] = (last + 0.02 * w) / 1.02; last = d[i]; d[i] *= 3.5;
      }
    }
  }
  return buf;
}

function playNoiseBed(type = 'brown', level = 0.15) {
  const src = ctx.createBufferSource();
  src.buffer = makeNoiseBuffer(type); src.loop = true;
  const filt = ctx.createBiquadFilter(); filt.type = 'lowpass';
  filt.frequency.value = 800; filt.Q.value = 0.5;
  const g = ctx.createGain(); g.gain.value = 0.0001;
  src.connect(filt).connect(g).connect(master);
  src.start();
  g.gain.exponentialRampToValueAtTime(level, ctx.currentTime + 4);
  return { src, filt, gain: g };
}
```

- **Brown noise** = the meditation-app favorite (deep, womb-like, masks the environment without hiss). **Pink** = "rain-adjacent". White is too harsh; avoid as a default.
- **Rain**: pink noise → highpass ~400 Hz + lowpass ~6000 Hz, plus optional random tiny "droplet" blips (short 2 kHz–4 kHz sine pings with 30 ms decay scheduled at Poisson-random times). Filtered pink alone already reads as steady rain.
- **Ocean waves**: brown/pink noise with a **slow LFO (~0.07–0.12 Hz, i.e., one wave per 8–14 s) on the lowpass cutoff AND on gain**, slightly offset in phase — cutoff sweeping 300→1400 Hz makes the "wash" while the gain swell makes the wave rise and retreat:

```js
const lfo = ctx.createOscillator(); lfo.frequency.value = 0.09;
const toCutoff = ctx.createGain(); toCutoff.gain.value = 500;    // ±500 Hz around base
filt.frequency.value = 850;
lfo.connect(toCutoff).connect(filt.frequency);
const toGain = ctx.createGain(); toGain.gain.value = 0.35 * level;
lfo.connect(toGain).connect(g.gain);
lfo.start();
```

Randomize wave period slightly (`lfo.frequency.setTargetAtTime(0.07+Math.random()*0.05, t, 4)` every ~15 s) so it never sounds mechanical.

### 1.7 Volume ramping, crossfading, mixing

- **Every user-facing volume slider** should drive a `GainNode` via `setTargetAtTime(value, now, 0.05)` — never set `.value` directly during playback (zipper noise).
- Perceptual mapping: sliders should be **quadratic/exponential** (`gain = slider^2`), not linear — linear sliders feel like everything happens in the bottom 20%.
- **Crossfade between presets**: run old and new voice graphs simultaneously for 2–4 s; equal-power crossfade (`gainA = cos(x·π/2)`, `gainB = sin(x·π/2)` implemented as two `setValueCurveAtTime` curves) avoids the mid-fade dip of linear crossfades.
- Recommended bus architecture for this app: `entrainmentBus`, `ambientBus`, `bellBus` → each with its own GainNode (= the 3 mix sliders) → `masterGain` → `DynamicsCompressorNode` → destination. Bells route with a small gain-duck on the entrainment bus if you want them to speak clearly (optional; the limiter usually suffices).

---

## 2. Mobile Constraints (the part that breaks naive implementations)

### 2.1 iOS Safari autoplay / unlock

- An `AudioContext` created outside a user gesture starts `suspended`. **Create or `resume()` it synchronously inside a real `touchend`/`click` handler** — the big "התחל" button IS your unlock. Don't `await` anything before calling `resume()`.
- On iOS the context can enter a nonstandard **`interrupted`** state (phone call, Siri, screen lock, app switch). `resume()` from a fresh user gesture usually recovers it, but there are documented cases where it sticks. Defensive pattern:

```js
document.addEventListener('visibilitychange', () => {
  if (!document.hidden && ctx && ctx.state !== 'running') ctx.resume();
});
ctx.onstatechange = () => { updatePlayButtonUI(ctx.state); };
// Nuclear option if resume() no-ops repeatedly: tear down and rebuild the whole
// context + graph on the next user tap. Since everything is synthesized from
// parameters, rebuild is cheap — architect voices as (params -> graph) factories.
```

- Also respect the **silent/ring switch**: Web Audio via `AudioContext` is treated as "ambient" and is muted by the hardware silent switch on iOS *unless* an `<audio>` element is also playing — one more reason for the trick below.

### 2.2 Background audio / screen lock on iOS (critical for meditation!)

Plain Web Audio **stops when the screen locks or Safari backgrounds** — fatal for a 20-minute eyes-closed session. The battle-tested workaround stack:

1. **Silent `<audio>` loop trick**: keep an `<audio loop playsinline>` element playing a tiny silent file (embed a ~1 s silent MP3/WAV as a base64 `data:` URI, a few hundred bytes). An actively playing media element promotes the page to a "media playback" session, which (on current iOS Safari, incl. Home-Screen PWAs in most versions) keeps the audio pipeline — including your AudioContext — alive under screen lock. Start it in the same user gesture. Caveat: Apple has repeatedly regressed/changed this behavior between iOS versions (17/18/26 threads on Apple forums document breakage windows); treat lock-screen playback as *best effort* and also do #3.
2. **Media Session API**: set metadata + handlers so the lock screen shows your session with working play/pause, and so iOS treats you as a media app:

```js
if ('mediaSession' in navigator) {
  navigator.mediaSession.metadata = new MediaMetadata({
    title: 'צלילה לתטא — 6Hz', artist: 'ThetaHealing Daily', album: 'Frequencies' });
  navigator.mediaSession.setActionHandler('play',  () => resumeSession());
  navigator.mediaSession.setActionHandler('pause', () => pauseSession());
  navigator.mediaSession.playbackState = 'playing';
}
```

3. **Honest UX fallback**: a setting "השאר מסך דולק בזמן תרגול" using Wake Lock (§3), defaulted ON for iOS, with screen dim via a CSS near-black overlay at minimal brightness cost (OLED). Tell users: "לחוויה עם מסך כבוי, הוסיפו למסך הבית / השאירו את הטלפון הפוך". Many web meditation apps ship exactly this compromise.
4. Android Chrome is far friendlier: Web Audio generally continues in background/screen-off, especially with Media Session active.

### 2.3 Other mobile quirks

- **Sample rate**: iOS may give you 44.1 k or 48 k depending on route (Bluetooth switches it!), and the context can come up at the "wrong" rate after a route change, sounding pitch-shifted — never hardcode `sampleRate`; derive all buffer math from `ctx.sampleRate`, and rebuild on `devicechange`/`statechange` weirdness.
- **Route changes** (unplugging headphones) pause/interrupt: listen for state changes and pause gracefully rather than blasting the speaker (also a safety point — see §6).
- **Battery**: oscillators + one looped noise buffer + a compressor is a tiny graph (<1–2% CPU). The battery hogs are the *screen* (mitigate with dim overlay) and doing DSP in `ScriptProcessorNode` (deprecated, main-thread, janky — don't; precomputed loop buffers or AudioWorklet only). Use `latencyHint: 'playback'`.
- **Do not use `OfflineAudioContext` renders on the fly for beds** on low-end phones mid-session; pre-generate buffers at init (the 10 s stereo noise buffer costs ~3.8 MB RAM at 48 k — fine).

---

## 3. Timer + Audio Integration

### 3.1 Scheduling interval bells: two clocks

`setTimeout`/`setInterval` drift badly (throttled to ≥1 Hz in background tabs, seconds-level drift over a 20-min session, GC pauses). `ctx.currentTime` is a sample-accurate hardware clock. Use **Chris Wilson's look-ahead scheduler** ("A Tale of Two Clocks"): a sloppy JS timer ticks every ~250 ms (bells don't need 25 ms granularity) and schedules any bell due in the next ~1 s at its exact audio-clock time:

```js
const session = { startTime: 0, bellEvery: 300 /*s*/, nextBell: 0, endAt: 0, timer: null };

function startSession(minutes, bellIntervalSec) {
  session.startTime = ctx.currentTime;
  session.endAt = session.startTime + minutes * 60;
  session.nextBell = session.startTime + bellIntervalSec;
  session.timer = setInterval(schedulerTick, 250);
  strikeBowl(ctx.currentTime + 0.05);                    // opening bell
}
function schedulerTick() {
  const ahead = ctx.currentTime + 1.0;                   // 1s lookahead
  while (session.nextBell < ahead && session.nextBell < session.endAt) {
    strikeBowl(session.nextBell);                        // sample-accurate
    session.nextBell += session.bellEvery;
  }
  if (ctx.currentTime >= session.endAt) endSession();    // triple bell + master fade
}
function endSession() {
  clearInterval(session.timer);
  [0, 4, 8].forEach(dt => strikeBowl(ctx.currentTime + dt, 440, 0.5)); // closing bells
  master.gain.setTargetAtTime(0.0001, ctx.currentTime + 10, 5);        // long fade-out
}
```

Because bells are scheduled on the *audio* clock, even if the JS timer gets throttled, any bell already scheduled fires perfectly; on wake, the `while` loop catches up. For the elapsed-time UI, also compute from `ctx.currentTime - session.startTime`, not from your own counters — and recompute on `visibilitychange` (don't trust interval ticks that never fired).

### 3.2 Wake Lock API

```js
let wakeLock = null;
async function keepAwake() {
  try {
    wakeLock = await navigator.wakeLock.request('screen');
    wakeLock.addEventListener('release', () => { wakeLock = null; });
  } catch (e) { /* denied (low battery etc.) — fine, just informational */ }
}
document.addEventListener('visibilitychange', () => {   // lock is auto-released on hide
  if (!document.hidden && sessionRunning) keepAwake();  // MUST re-request on return
});
```

Supported in Safari ≥16.4, Chrome, Firefox ≥126 — safe to use, always feature-detect. Release it in `endSession()`. Pair with a "dim" mode: full-screen black overlay at `opacity:.96` with a tiny remaining-time readout — keeps the wake lock valid while nearly eliminating OLED drain.

---

## 4. What Leading Apps Do (patterns worth copying)

| App | Core model | Notable patterns to borrow |
|---|---|---|
| **Brainwave / Brain Waves (Banzai/rbx)** | 35+ pure binaural programs | Sells **outcomes, not Hz** ("Deep Sleep", "Creativity Boost"); 3-layer mix = entrainment + ambient + optional music, each with own slider; multi-stage programs that *ramp* the beat frequency over the session (e.g., 10→6 Hz over 8 min for a "descent") |
| **Brain.fm** | Patented amplitude "neural phase-locking" modulation (essentially sophisticated isochronic AM inside real music) | Session types = Focus / Relax / Sleep with duration first; strong onboarding claim + "works in ~5 min"; modulation depth as a user-facing "neural effect: low/med/high" setting |
| **BetterSleep (ex-Relax Melodies)** | Layered sound **mixer** (up to ~8 sounds), binaural/isochronic as just another layer | The mixer UI is the killer feature: tap tiles to toggle layers, per-layer volume; save named custom mixes; gentle fade-out sleep timer |
| **Insight Timer** | Timer-first: interval bells + ambient | Preset = duration + bell type + bell intervals + ambient bed, saved as one-tap favorites; multiple bowl timbres; "strike at start / intervals / end ×3" convention |
| **Atmosphere / myNoise-style** | Generative soundscape mixers | Slider-per-band noise shaping; calibrate-to-your-hearing step |

Cross-app conventions that should be treated as **standards**:
- **Never expose raw Hz as the primary UI.** Primary selection = target state (שינה / רגיעה עמוקה / תטא / ריכוז / מדיטציה), with Hz shown as secondary "geek info" and an advanced screen for manual tuning.
- **Fade-in 2–5 s, fade-out 5–30 s** (sleep presets fade out for as long as 60 s). Never hard-cut entrainment audio.
- **Frequency ramps**: quality programs *glide* the user down (start alpha ~10 Hz, ramp to theta 5–6 Hz over 3–10 min) because meeting the brain near its current state entrains better than jumping straight to the target. Implement with `oscR.frequency.linearRampToValueAtTime(carrier + targetBeat, t + rampSec)` (binaural) or LFO frequency ramp (isochronic). This is a cheap, high-credibility feature.
- Default session lengths: 10 / 15 / 20 / 30 / 45 / ∞, with 15–20 min the default for meditation, 30–45 for sleep-onset.
- Entrainment layer is usually mixed **below** the ambient layer (beat at ~20–40% of bed volume) — the beat should be *felt at the edge of attention*, not foregrounded.

---

## 5. Recommended Preset Architecture for the App

### 5.1 Data model — everything is parameters

```js
const PRESETS = [
  { id:'theta-dive',  name:'צלילה לתטא',   icon:'🌀',
    entrain:{ type:'binaural', carrier:200, beat:6.0, rampFrom:10, rampSec:300, level:0.22 },
    fallback:{ type:'isochronic' },                    // auto-swap if user taps "בלי אוזניות"
    ambient:{ type:'brown', lowpass:700, level:0.30 },
    bells:{ start:true, intervalSec:0, end:3 },        // bell at start, 3 strikes at end
    duration:20, fadeIn:4, fadeOut:15, headphones:true },

  { id:'deep-sleep',  name:'שינה עמוקה',   icon:'🌙',
    entrain:{ type:'isochronic', carrier:150, beat:2.5, rampFrom:6, rampSec:600, level:0.18, lfoShape:'smooth' },
    ambient:{ type:'ocean', level:0.35 },
    bells:{ start:false, end:0 },
    duration:45, fadeIn:6, fadeOut:60, headphones:false },

  { id:'focus',       name:'ריכוז',        icon:'🎯',
    entrain:{ type:'binaural', carrier:250, beat:10, level:0.20 },
    ambient:{ type:'pink', highpass:300, lowpass:5000, level:0.25 },  // "rain"
    duration:30, fadeIn:3, fadeOut:8, headphones:true },

  { id:'calm-alpha',  name:'רוגע',         icon:'🍃',
    entrain:{ type:'isochronic', carrier:180, beat:8, level:0.18, lfoShape:'smooth' },
    ambient:{ type:'brown', level:0.3 }, duration:15, headphones:false },

  { id:'theta-heal',  name:'ThetaHealing תרגול', icon:'✨',   // signature preset
    entrain:{ type:'binaural', carrier:200, beat:5.5, rampFrom:9, rampSec:240, level:0.22 },
    solfeggio:{ freq:528, level:0.10 },                 // 528 layered softly under it
    ambient:{ type:'brown', level:0.25 },
    bells:{ start:true, intervalSec:420, end:3 },       // interval bell every 7 min
    duration:20, headphones:true },
];
// Solfeggio picker (separate tab): 9 chips
const SOLFEGGIO = [
  {f:174, name:'שחרור כאב'},   {f:285, name:'התחדשות'},   {f:396, name:'שחרור פחד'},
  {f:417, name:'שינוי'},        {f:528, name:'אהבה / DNA'}, {f:639, name:'מערכות יחסים'},
  {f:741, name:'ניקוי'},        {f:852, name:'אינטואיציה'}, {f:963, name:'הארה'},
];
```

Brainwave band reference for the advanced screen: Delta 0.5–4 Hz (שינה), Theta 4–8 Hz (מדיטציה עמוקה, מצב תטא — **6–7 Hz הוא היעד הקלאסי של ThetaHealing**), Alpha 8–13 Hz (רוגע ערני), Beta 14–30 Hz (ריכוז), Gamma 30–50 Hz (40 Hz הוא הסטנדרט המחקרי).

### 5.2 UI structure (mobile RTL)

1. **מסך ראשי**: grid of preset cards (state name + icon + duration). Tap → session screen.
2. **מסך סשן**: big play/pause, remaining time ring, 3 sliders (תדר / רקע / עוצמה כללית), bell toggle, duration chips (10/15/20/30/45/∞), headphone banner if `preset.headphones`.
3. **טאב סולפג'יו**: 9 chips; selecting plays the harmonically-sweetened drone (§1.4); allow combining with an ambient bed and optional slow tremolo.
4. **מתקדם**: raw Hz controls, binaural↔isochronic switch, carrier slider (100–400 Hz), LFO shape, ramp settings.
5. Persist last mix + custom presets to `localStorage`.
6. Session engine = pure function `buildGraph(preset)` returning `{start(), stop(), setLevel(bus, v)}` — this also makes the iOS "rebuild context" recovery (§2.1) trivial.

Auto-stop = §3.1 `endSession()`: closing bells, long master fade, release wake lock, `mediaSession.playbackState='paused'`.

---

## 6. Safety UX (small module, real liability coverage)

Show once on first launch (+ always accessible in settings), and inline where relevant:

- **עוצמת שמע**: start every session at a conservative default (master 0.5 ≈ −12 dB); on first-ever play show "התחילו בעוצמה נמוכה והעלו בהדרגה". If the user pushes master above ~85%, show a transient "🔊 עוצמה גבוהה עלולה לפגוע בשמיעה בהאזנה ממושכת". The `DynamicsCompressorNode` limiter is your technical backstop against synth spikes.
- **אוזניות לבינאורלי**: banner on binaural presets — "🎧 אפקט בינאורלי דורש אוזניות (סטריאו). בלי אוזניות? נעבור אוטומטית לגלים איזוכרוניים" with the one-tap fallback.
- **לא בזמן נהיגה**: "אין להאזין לתדרי הרפיה בזמן נהיגה או הפעלת מכונות" — entrainment toward theta/delta is explicitly drowsiness-inducing; every serious entrainment product carries this warning.
- **אפילפסיה**: rhythmic photic *and* auditory stimulation carries a (small, debated) theoretical seizure-trigger concern; the industry-standard warning: "הסובלים מאפילפסיה, נוטלי תרופות אנטי-אפילפטיות, קוצב לב, הריון — יש להיוועץ ברופא לפני שימוש בגירוי קצבי". Keep any *visual* pulsing in the UI slower than 3 Hz or static.
- **טינטון ורגישות שמיעתית**: pure tones and AM pulses can aggravate tinnitus/hyperacusis for some users — "אם מופיעים צפצוף, אי-נוחות או כאב — הפסיקו את השימוש". Offering the ambient-only mode (bed without entrainment) is the graceful degradation.
- **מסגור כללי**: "כלי לרגיעה ותרגול — אינו טיפול רפואי ואינו תחליף לייעוץ רפואי" (also honest: the clinical evidence for binaural-beat entrainment is mixed; the relaxation value is real, the medical claims should stay soft).
- **Route-change protection**: on headphone unplug (context `statechange`/`interrupted`), auto-pause instead of continuing on the loudspeaker.

---

## 7. Implementation Checklist (ordered)

1. `initAudio()` inside the start-button gesture: context + master + limiter + silent `<audio>` loop + Media Session + Wake Lock.
2. Voice factories: `playTone`, `createBinaural`, `createIsochronic`, `solfeggioVoice`, `strikeBowl`, `playNoiseBed` (+ ocean LFO) — all writing into 3 buses.
3. Look-ahead scheduler for bells + session end; UI time derived from `ctx.currentTime`.
4. Preset table (§5.1) → `buildGraph(preset)`; equal-power crossfade on preset switch; frequency ramp support.
5. iOS resilience: `visibilitychange` resume, `statechange` UI sync, full-rebuild fallback, never hardcode sample rate.
6. Safety banners + headphone/isochronic fallback + quadratic volume sliders.
7. QA on real iPhone (Safari + installed PWA): screen lock during session, silent switch, phone-call interruption, Bluetooth route change.

---

**Sources**: [WebAudio spec issue — AudioContext stuck "interrupted" on Safari](https://github.com/WebAudio/web-audio-api/issues/2585) · [MDN — BaseAudioContext.state](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/state) · [Unlock Web Audio in Safari/iOS — Matt Montag](https://www.mattmontag.com/web/unlock-web-audio-in-safari-for-ios-and-macos) · [Apple Dev Forums — iOS audio lockscreen problem in PWA](https://developer.apple.com/forums/thread/762582) · [Apple Dev Forums — PWA playback broken after iOS 26.0.1](https://developer.apple.com/forums/thread/805900) · [web.dev — A Tale of Two Clocks (audio scheduling)](https://web.dev/articles/audio-scheduling) · [DEV — Why JS timers drift; Web Audio metronome](https://dev.to/kandz/why-javascript-timers-drift-building-a-high-precision-metronome-with-web-audio-api-c0a) · [MDN — Screen Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API) · [Chrome Developers — Wake Lock](https://developer.chrome.com/docs/capabilities/web-apis/wake-lock) · [Noisehack — Generate noise with Web Audio API](https://noisehack.com/generate-noise-web-audio-api/) · [Sonoport — Synthesising sounds with Web Audio](https://sonoport.github.io/synthesising-sounds-webaudio.html) · [Sonoport — Understanding the Web Audio clock](https://sonoport.github.io/web-audio-clock.html) · [BinauralBeatJS](https://github.com/ichabodcole/BinauralBeatJS) · [PureBinaural — isochronic generator](https://purebinaural.com/isochronic-tones-generator) · [Wikipedia — Isochronic tones](https://en.wikipedia.org/wiki/Isochronic_tones) · [Wikipedia — Banded waveguide synthesis (bowls/bells)](https://en.wikipedia.org/wiki/Banded_waveguide_synthesis) · [frequency.technology — Binaural beats apps 2026](https://frequency.technology/binaural-beats-apps/) · [Insight Timer — meditation presets](https://help.insighttimer.com/support/solutions/articles/67000665012-how-can-i-create-a-meditation-preset-) · [Brain.fm review](https://www.timeatlas.com/brain-fm-review/) · [Insight Timer — binaural beats](https://insighttimer.com/techniques/sound-healing/binaural-beats)