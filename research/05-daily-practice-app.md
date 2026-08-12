# ThetaHealing Daily Practice App — Content Architecture Research Report

**Target:** Hebrew-first, mobile-first, single-file HTML/PWA daily companion for ThetaHealing (תטא הילינג) practitioners & students.
**Scope note:** This is a *practice companion*, not a substitute for certified training. thetahealing.com actively protects its IP (seminars, manuals, "1,000+ downloads" are paid course material) — the app should ship *original, style-consistent* content, use the ® disclaimer ("אינו קשור רשמית ל-THInK / Vianna Stibal"), and avoid reproducing manual text verbatim.

---

## 1. What a Daily ThetaHealing Practice Contains

Synthesized from official ThetaHealing sources, instructor sites (Dawn Maree, Dr. Lisa Cooney, healingwiththeta.com, Israeli teachers: rmalul.com, oripui.com, nogabenisrael.com, ofenbari.co.il, reidman.co.il), and the books *ThetaHealing*, *Advanced ThetaHealing*, *Seven Planes of Existence*, *Digging for Beliefs*.

### 1.1 The core daily unit: Roadmap to the 7th Plane (מפת הדרכים למישור השביעי)
The signature meditation, done ideally every morning. Canonical sequence:
1. **Ground/center** — sit, breathe deeply, imagine energy coming up from the center of the Earth through the feet, up through all chakras to the crown (some lineages now skip the earth phase and "go up" directly — both variants exist; app should offer both).
2. **Go up** — visualize yourself in a ball of light rising above your space (מעל המרחב שלך), above the room, the sky, the planet.
3. **Through the universe** — past stars/galaxies to the edge of the universe.
4. **Through the layers of light** — white light → dark light → white light → golden light → thick jelly-like substance (the "Laws", חוקי הבריאה) → iridescent pearly-white light of the Seventh Plane (אור לבן-צחור מנצנץ).
5. **Dissolve into All That Is** — "you are part of the energy of Creation"; command/request wording: "בורא כל היש, מצווה/מבקש/ת..." ("Creator of All That Is, it is commanded/requested...").
6. **Do the work** — witnessing healings, downloads, belief replacements, manifesting — always *as witness*, the Creator does the work.
7. **Return & ground** — rinse in white light, return awareness to body, ground into the earth, open eyes. Key teaching: "each time you practice you go deeper into theta — you are training your brain."
Typical duration: 5–15 min guided; experienced practitioners drop to ~30 seconds ("crown push").

### 1.2 Daily downloads (הורדות יומיות)
"Downloads" = feelings/knowings instilled from the Creator, phrased as "I know what it feels like to… / I know how to… / I know that it's possible… / I am worthy of…" — always **with verbal consent ("כן")**. Vianna herself publishes *Daily Meditations and Downloads*; instructors run daily-download WhatsApp/Facebook groups. Daily unit: go to 7th plane → receive 3–10 themed downloads → accept each with "כן" → gratitude. This maps perfectly to a "download of the day" card mechanic.

### 1.3 Belief work & digging journaling (עבודת אמונות וחפירה)
From *Digging for Beliefs*: digging finds the **bottom belief** (אמונת השורש/התחתית) that holds a whole stack in place. Four belief levels: **Core (גרעיני), Genetic (גנטי), History (היסטורי), Soul (נשמתי)**. Ladder questions: "מה היה קורה אם...?", "מה הכי גרוע ב...?", "מתי זה התחיל?", "מה זה נותן לך? / איך זה משרת אותך?", "ממי למדת את זה?". Bottom belief is recognized by an "aha"/emotional charge and verified by muscle test. Then release + replace with Creator's perspective + download the new feeling. Daily-app version: a guided journaling flow (10–20 min, 2–3×/week).

### 1.4 Muscle testing (בדיקת שרירים)
Used to verify programs before/after work. Two self-test methods to teach in-app:
- **Standing sway**: feet shoulder-width, say "כן, כן, כן" (body tips forward) / "לא, לא, לא" (tips back); calibrate with "שמי X" true/false.
- **Finger loop/pull** method as alternative.
Critical rule from all sources: **drink water first** — dehydration invalidates results. App can include a 2-minute daily calibration drill + hydration reminder.

### 1.5 Manifesting & gratitude (הבעת תודה ומניפסטציה)
From the Manifesting & Abundance seminar: manifest **from the 7th plane** (not from ego/3rd plane), state the wish in present tense, "בדרך הטובה והמיטבית ביותר" (highest and best way), visualize it done, feel gratitude, release attachment to timing (divine timing). Daily gratitude: 3 things morning ("מה אני מזמין/ה היום") + evening review.

### 1.6 Evening practice
Instructor-recommended close of day: gratitude review → forgiveness/release of the day's charges (mini-digging if something triggered) → optional short 7th-plane visit → sleep meditation (theta is the pre-sleep brainwave, so bedtime is a natural theta window — good marketing/UX hook).

**⇒ Canonical daily loop for the app:**
בוקר: עלייה למישור השביעי (5–15 ד׳) + הורדת היום + כוונה/מניפסטציה (2 ד׳) | צהריים: איפוס 3 דקות | ערב: תודה (3 פריטים) + חפירה/יומן (2–3 בשבוע) + מדיטציית שינה.

---

## 2. Survey of Daily-Practice Mechanics in Leading Apps

| App | Mechanics worth copying | Notes |
|---|---|---|
| **Calm** | "Daily Calm" — ONE fresh flagship session/day; post-session emoji **mood check-in** + monthly mood calendar; gratitude + sleep logging; sleep stories | The single-daily-anchor model fits "הורדת היום" perfectly |
| **Headspace** | Structured **Basics course** (10×10 min) as mandatory-feeling onboarding; morning/afternoon/evening contextual recommendations; mood check-in *before and after* with stressor tags; streaks + milestone badges; SOS 3-min sessions | Copy: onboarding path + time-of-day home screen + SOS quick reset |
| **Insight Timer** | Best-in-class **unguided timer**: duration, start/interval/end bells (10 bell types × 1-3 strikes), 19 ambient sounds, **saved presets**, meditation journal; partial sessions still count toward streak (anti-"what-the-hell effect"); community "X people meditating now" | Copy: preset timer with bells for self-guided 7th-plane work; lenient streak counting |
| **Waking Up** | 28-day intro course pairing **daily practice + daily theory lesson**; Daily Meditation + Daily "Moment" (micro-reflection push); theory/practice/timer three-tab IA | Copy: pair each practice day with a 1-min ThetaHealing concept card (planes, belief levels, etc.) |
| **5 Minute Journal / Gratitude apps** | Morning intentions + evening reflection split; prompts + affirmations; streaks; photo attachment; daily quote rotation | Copy: AM/PM journal split — maps to intention/gratitude |
| **Habit apps (Duolingo-school analyses)** | Streak **freezes/repair**, weekly goals instead of daily perfection, milestone celebrations (7/21/30/100 days), "completable on a bad day" minimum action (10-sec, not 20-min) | Key warning: streaks cause shame/burnout in wellness contexts; a break must be a *recoverable event, not a hard reset* |

**Consensus stack to implement:** daily anchor session + streak-with-grace + total-minutes/session stats + mood before/after + saved timer presets + a structured starter course + evening journal + gentle local-notification reminders.

---

## 3. Content Catalog to Ship

### 3.1 Guided meditation scripts (10 outlines, Hebrew narration)

1. **עלייה למישור השביעי — המסע המלא** (12–15 ד׳): הרפיה ונשימה (2) → אנרגיית האדמה דרך הצ׳אקרות (2) → עלייה מעל המרחב, כדור הארץ, היקום (3) → שכבות האור: לבן, כהה, זהוב, החומר הג׳לטיני, האור הצחור (3) → התמזגות ב"כל מה שיש" + השהיה שקטה (3) → חזרה והארקה (2). פעמון בכניסה למישור השביעי.
2. **עלייה מהירה — דחיפת הכתר** (3–4 ד׳): למתרגלים מנוסים; נשימה אחת → "בורא כל היש" → עלייה ישירה → שהייה 2 ד׳ → חזרה.
3. **הארקה ומרכוז** (5–7 ד׳): שורשים למרכז האדמה, איסוף אנרגיה חזרה מהיום, איזון צ׳אקרות, מעטפת אור.
4. **פתיחת הלב ואהבת הבורא** (10 ד׳): עלייה → בקשה לחוות את אהבת הבורא ללא תנאי → הורדות אהבה עצמית → עיגון בלב.
5. **שפע ומניפסטציה מהמישור השביעי** (12 ד׳): עלייה → ניסוח הבקשה בזמן הווה "בדרך הטובה והמיטבית" → צפייה בחזון כמושלם → תודה → שחרור לתזמון אלוהי.
6. **ריפוי ואנרגיה לגוף** (10 ד׳): עלייה → בקשת עדות לריפוי → סריקת גוף באור → הורדות בריאות.
7. **ניקוי וריענון אנרגטי** (7 ד׳): ניתוק חיבורים אנרגטיים של היום, החזרת אנרגיה, מקלחת אור לבן.
8. **איפוס 3 דקות (SOS)** (3 ד׳): נשימה 4-4-8 → עלייה מהירה → משפט אחד: "אני מוקף/ת באור הבורא" → חזרה. חייב להיות נגיש בלחיצה אחת מהמסך הראשי.
9. **מדיטציית ערב — תודה ושחרור** (8 ד׳): סקירת היום בעיני הבורא → 3 תודות → סליחה ושחרור מטענים → הכנה לשינה.
10. **מדיטציית שינה — צלילה לתטא** (15–20 ד׳, קול איטי + רעש רקע): הרפיה מתקדמת מכף רגל לראש → עלייה איטית → שהייה באור → ללא חזרה מודעת (נרדמים בתוכה).

(אופציונליים: מדיטציית סליחה ממוקדת; מדיטציית "היום המושלם שלי" לבוקר — 5 ד׳ של בריאת היום מראש.)

### 3.2 Download library — ~80 examples, ThetaHealing phrasing, gendered toggle (אני יודע/ת)

**App mechanic:** show 3–5/day; user taps "כן ✓" to accept each (mirrors verbal-consent doctrine); accepted downloads log to history. Provide זכר/נקבה phrasing toggle at onboarding.

**אהבה עצמית (10)**
- אני יודע/ת איך זה מרגיש לאהוב את עצמי ללא תנאי.
- אני יודע/ת איך לחיות את היומיום שלי מתוך אהבה עצמית.
- אני יודע/ת שאני ראוי/ה לאהבה בדיוק כפי שאני.
- אני יודע/ת איך זה מרגיש להיות שלם/ה עם מי שאני.
- אני יודע/ת איך להציב גבולות מתוך אהבה, בלי אשמה.
- אני יודע/ת איך זה מרגיש לסלוח לעצמי בקלות.
- אני יודע/ת שמותר לי לנוח בלי להרגיש אשמה.
- אני יודע/ת איך לראות את עצמי דרך עיני הבורא.
- אני יודע/ת איך זה מרגיש להיות גאה בדרך שעברתי.
- אני יודע/ת שאני מספיק/ה, בדיוק כמו שאני.

**שפע ופרנסה (10)**
- אני יודע/ת איך זה מרגיש לחיות בשפע בכל תחומי חיי.
- אני יודע/ת איך לקבל כסף בשמחה ובקלות.
- אני יודע/ת שאפשר להרוויח בשפע מעשייה שאני אוהב/ת.
- אני יודע/ת איך זה מרגיש שכל צרכיי מסופקים תמיד.
- אני יודע/ת איך לחיות בלי פחד ממחסור.
- אני יודע/ת שכסף הוא אנרגיה נקייה, ומותר לי ליהנות ממנו.
- אני יודע/ת איך זה מרגיש להיות ראוי/ה להצלחה.
- אני יודע/ת איך לזהות הזדמנויות שהבורא שולח לי.
- אני יודע/ת איך לתת ולקבל בזרימה מאוזנת.
- אני יודע/ת שהשפע שלי אינו בא על חשבון איש.

**זוגיות ומערכות יחסים (10)**
- אני יודע/ת איך זה מרגיש לאהוב ולהיות נאהב/ת בביטחון מלא.
- אני יודע/ת איך לחיות בזוגיות מתוך כבוד הדדי.
- אני יודע/ת איך זה מרגיש לסמוך על בן/בת הזוג שלי.
- אני יודע/ת שאפשר לאהוב בלי לאבד את עצמי.
- אני יודע/ת איך לתקשר את צרכיי בבהירות וברוך.
- אני יודע/ת איך זה מרגיש להיות מובן/ת ומוערך/ת.
- אני יודע/ת איך לשחרר קשרים שאינם משרתים אותי, באהבה.
- אני יודע/ת שמגיעה לי אהבה נאמנה ויציבה.
- אני יודע/ת איך זה מרגיש לחיות בלי פחד מנטישה.
- אני יודע/ת איך לראות את הטוב באנשים ולשמור על עצמי בו-זמנית.

**בריאות וגוף (10)**
- אני יודע/ת איך זה מרגיש לחיות בגוף בריא ומלא חיות.
- אני יודע/ת איך לאהוב את הגוף שלי ולהודות לו.
- אני יודע/ת שהגוף שלי יודע לרפא את עצמו.
- אני יודע/ת איך זה מרגיש לישון שינה עמוקה ומרפאת.
- אני יודע/ת איך להזין את גופי במה שטוב עבורו.
- אני יודע/ת איך לחיות בלי כאב מהעבר שאגור בגוף.
- אני יודע/ת איך זה מרגיש לקום בבוקר עם אנרגיה ושמחה.
- אני יודע/ת שמותר לי להיות בריא/ה גם כשסביבי חולי.
- אני יודע/ת איך להקשיב לאותות של הגוף שלי בזמן.
- אני יודע/ת איך זה מרגיש להזדקן בחן, בבריאות ובשמחה.

**ביטחון והגנה (10)**
- אני יודע/ת איך זה מרגיש להיות מוגן/ת ובטוח/ה בעולם.
- אני יודע/ת איך לחיות בלי פחד מהעתיד.
- אני יודע/ת שאני מוקף/ת באור הבורא בכל רגע.
- אני יודע/ת איך זה מרגיש לבטוח בתהליך של החיים.
- אני יודע/ת איך להישאר רגוע/ה גם בתוך אי-ודאות.
- אני יודע/ת שמותר לי לבלוט ולהיראות בביטחון.
- אני יודע/ת איך לחיות בלי צורך להיות דרוך/ה כל הזמן.
- אני יודע/ת איך זה מרגיש שהעולם הוא מקום ידידותי עבורי.
- אני יודע/ת איך להבחין בין אינטואיציה לבין פחד.
- אני יודע/ת שאני יכול/ה לעמוד בכל מה שהחיים מביאים.

**סליחה ושחרור (10)**
- אני יודע/ת איך זה מרגיש לסלוח באמת ולהשתחרר.
- אני יודע/ת איך לשחרר כעס בלי לפגוע בעצמי או באחרים.
- אני יודע/ת שסליחה אינה הסכמה למה שקרה — היא חופש שלי.
- אני יודע/ת איך לשחרר את העבר ולחיות בהווה.
- אני יודע/ת איך זה מרגיש לחיות בלי טינה.
- אני יודע/ת איך לשחרר אשמה ולקחת אחריות מתוך חמלה.
- אני יודע/ת איך להשלים עם מה שאי אפשר לשנות.
- אני יודע/ת איך זה מרגיש להתחיל דף חדש.
- אני יודע/ת איך לשחרר אמונות שירשתי ואינן שלי.
- אני יודע/ת איך זה מרגיש להיות קל/ה וחופשי/ה.

**הורות ומשפחה (10)**
- אני יודע/ת איך זה מרגיש להיות הורה מתוך רוגע ואהבה.
- אני יודע/ת איך להציב גבולות לילדיי מתוך חיבור ולא מתוך כעס.
- אני יודע/ת שאני הורה טוב/ה גם כשאני טועה.
- אני יודע/ת איך לרפא דפוסים משפחתיים ולא להעביר אותם הלאה.
- אני יודע/ת איך זה מרגיש לכבד את הוריי ולהישאר נאמן/ה לעצמי.
- אני יודע/ת איך לאהוב את משפחתי בלי לשאת את הכאב שלה.
- אני יודע/ת איך זה מרגיש בית שקט ומלא אהבה.
- אני יודע/ת איך לתת לילדיי שורשים וכנפיים.
- אני יודע/ת שמותר לי לבקש עזרה בהורות.
- אני יודע/ת איך ליהנות מהרגעים הקטנים עם משפחתי.

**חיבור לבורא ורוחניות (10)**
- אני יודע/ת איך זה מרגיש להיות מחובר/ת לבורא כל היש.
- אני יודע/ת איך לעלות למישור השביעי בקלות ובבהירות.
- אני יודע/ת איך להבחין בקול הבורא מבין שאר הקולות.
- אני יודע/ת שאני חלק מ"כל מה שיש" ולעולם איני לבד.
- אני יודע/ת איך לקבל תשובות בבהירות ובאהבה.
- אני יודע/ת איך לחיות את היומיום שלי מתוך תדר גבוה.
- אני יודע/ת איך זה מרגיש לבטוח בתזמון האלוהי.
- אני יודע/ת איך להיות עד/ה לריפוי בענווה ובתודה.
- אני יודע/ת שמותר לי לקבל שפע רוחני וגשמי כאחד.
- אני יודע/ת איך זה מרגיש לחיות בהשראה יומיומית.

### 3.3 Digging-journal template (יומן חפירה)

Guided wizard, one screen per step, saved to localStorage:
1. **טריגר (מה קרה?)** — free text + emotion picker + intensity 1–10.
2. **אמונת פני השטח** — "מה המשפט שרץ לי בראש?" (e.g., "אני לא מספיק טוב/ה").
3. **בדיקת שרירים** — optional checkbox: נבדק? כן/לא (תוצאה).
4. **שאלות הסולם** (loop, add rows): מה היה קורה אם...? / מה הכי גרוע בזה? / מתי הרגשתי כך לראשונה? / מה זה נותן לי? / ממי למדתי את זה? — each answer becomes the next rung.
5. **אמונת התחתית** — highlight the sentence with the emotional charge; tag level: גרעיני / גנטי / היסטורי / נשמתי.
6. **מה אני לומד/ת מזה? (החוכמה)** — the lesson to keep (ThetaHealing never deletes without extracting the learning).
7. **אמונה חלופית + הורדה** — replacement phrased as download; button "קיבלתי — כן ✓"; optional re-test.
8. **סגירה** — intensity re-rating (before/after delta shown), gratitude line.
Journal list view: filterable by theme/level; "אמונות שהוחלפו" trophy shelf.

### 3.4 Weekly structure & starter programs

**שבוע טיפוסי (מסלול "מתרגל"):**
| יום | בוקר | ערב |
|---|---|---|
| א׳ | מסע מלא למישור השביעי + כוונה שבועית | תודה |
| ב׳ | עלייה מהירה + הורדות שפע | יומן חפירה |
| ג׳ | עלייה מהירה + הורדות אהבה עצמית | תודה + הארקה |
| ד׳ | פתיחת לב | יומן חפירה |
| ה׳ | מניפסטציה מהמישור השביעי | תודה |
| ו׳ | ריפוי הגוף / ניקוי אנרגטי | מדיטציית שינה |
| ש׳ | מנוחה או מדיטציה חופשית (טיימר) | סקירת שבוע + תודה |

**תוכנית 21 יום — "המסע למישור השביעי" (starter):** graduated durations (wk1: 5–10 ד׳, wk2: 10–15 ד׳, wk3: 15–20 ד׳), each day = practice + 1-min theory card (Waking Up model):
- **שבוע 1 — היסודות:** ימים 1–2 נשימה והארקה; 3–5 המסע המלא (מודרך); 6 בדיקת שרירים וכיול; 7 סיכום + הורדות ראשונות.
- **שבוע 2 — עבודת אמונות:** רמות האמונה, חפירה ראשונה מודרכת, סליחה, הורדות יומיות לפי תמה מתחלפת.
- **שבוע 3 — בריאה:** מניפסטציה, שפע, "היום המושלם שלי", עלייה מהירה עצמאית, יום 21 = טקס סיום + תעודת מסע.

**תוכנית 30 יום — "חיים בתדר תטא" (continuation):** 4 תמות שבועיות: אהבה עצמית → שפע → מערכות יחסים → ייעוד; each week = 5 practice days + 1 digging day + 1 rest/free-timer day (rest day *counts* toward completion — anti-shame).

### 3.5 Theory micro-cards (bonus content, 1 min each)
שבעת מישורי הקיום (מינרלים→צמחים→בע"ח→בני אדם→נשמות→חוקים→בורא), גלי מוח ותטא, 4 רמות אמונה, כללי ההורדה והסכמה חופשית, עד מול מרפא, תזמון אלוהי.

---

## 4. Gamification & Retention — Spiritual-App-Appropriate

**Do:**
- **רצף עדין (gentle streak):** count *any* practice ≥1 min (Insight Timer's partial-credit rule); 1 automatic "יום חסד" (grace day) per week; a broken streak shows "חזרת! ההתמדה שלך נשמרת" and restores after 2 consecutive days (repair, not reset). Frame as "ימי תרגול החודש: 18" (cumulative) more prominently than consecutive count — the break must be a recoverable event.
- **Weekly goal, not daily perfection:** default 5/7 days = "שבוע מלא".
- **Milestones, not badges-spam:** 7 / 21 / 30 / 100 / 365 ימי תרגול; מדיטציה מס׳ 50; אמונה מוחלפת מס׳ 10 — celebrated with a lotus/light animation + a special download as the reward (reward *is* content, on-brand).
- **Progress stats:** סך דקות בתטא, מסעות למישור השביעי, הורדות שהתקבלו, אמונות שהוחלפו — framed as a "מסע" map (path through the 7 planes as level metaphor for the 21-day course).
- **Mood/energy check-in** before & after (1-tap, 5 emoji + optional "תדר" slider 1–10): show the before→after delta — this is the single most convincing personal-proof mechanic (validated in the PMC study of Calm's mood check-in: associated with higher engagement).
- **Reminders:** user-set morning & evening local notifications with soft copy ("הבורא מחכה ☁️" / "3 דקות של אור לפני השינה?"); default OFF for midday.

**Don't (shame mechanics to exclude):** loss-aversion push ("אל תאבד את הרצף!"), leaderboards, guilt copy, red badges, streak-zero screens, paywall pressure mid-meditation. The minimum daily action must be completable on a bad day — that's the 3-minute SOS reset, or even just accepting one download ("הורדה אחת = יום תרגול").

---

## 5. UX for a Hebrew RTL Meditation PWA

- **RTL:** `<html dir="rtl" lang="he">`; use CSS logical properties (`margin-inline-start`, `padding-inline-end`) everywhere; mirror navigation/back arrows and list chevrons. **Exceptions (per Material Design bidirectionality guidance): timers, media progress bars, and numbers stay LTR in Hebrew** — a countdown "12:45" and the audio scrubber should run left→right; force `dir="ltr"` + `unicode-bidi: isolate` on those elements. Digits: standard Western numerals.
- **Typography:** Heebo / Assistant / Rubik (Google Fonts, subset+inlined as woff2 data-URI for single-file constraint); 1.6+ line-height; generous size (18px base) — meditation apps are read with tired eyes.
- **Palette:** dark-first "calm night" palette (deep indigo/midnight `#0f1226`–`#1a1f3d`, soft gold/pearl accents for the 7th-plane motif — לבן-צחור/זהב, lavender secondary); light theme optional. Avoid pure black (#000) and pure white text; use warm off-whites. Respect `prefers-color-scheme` but let the user pin dark (evening use dominates).
- **Timer UI:** Insight-Timer-style preset builder — duration, צלצול פתיחה/סיום, פעמוני ביניים (e.g., bell when "entering the 7th plane" at min 4), ambient loop (rain/om/silence). Bells via Web Audio API (synthesize or tiny base64 samples — keeps single-file). Circular progress ring, huge central time, one-tap pause. **Screen wake-lock** (`navigator.wakeLock`) during sessions; keep audio alive with a silent `<audio>` loop trick for iOS backgrounding.
- **Audio/TTS:** three tiers — (1) **Guided text mode**: auto-advancing script cards with timing (works 100% offline, zero audio weight); (2) **Web Speech API TTS** (`speechSynthesis`, `lang: 'he-IL'`): supported in Chrome/Safari/Samsung Internet; Hebrew voice availability depends on OS TTS engine (iOS has Carmit; Android via Google TTS) — always feature-detect, list `getVoices()` filtered to `he`, offer rate 0.75–0.85 (slow) and pitch controls, and gracefully fall back to text mode; (3) future: recorded MP3s by a real instructor (optional streamed layer).
- **Offline/PWA:** single HTML file with inline manifest (base64) + inline service worker via Blob URL, or manifest-less "Add to Home Screen" with `apple-mobile-web-app-capable`; **all state in localStorage** (settings, streaks, journal, accepted downloads, program progress) under one namespaced JSON key with schema-version field; export/import backup as downloadable JSON (journals are emotionally valuable — data loss is trust loss). Total budget: keep under ~1.5 MB so it loads on 3G.
- **Hebrew-specific content UX:** gender toggle (זכר/נקבה/ניטרלי "אני יודע.ת") applied app-wide via template tokens; keep sacred terms consistent — בורא כל היש, המישור השביעי, הורדות, עבודת אמונות, חפירה, בדיקת שרירים, הארקה (matching the Israeli teaching vocabulary).
- **Session flow pattern:** check-in (1 tap) → practice (full-screen, chrome hidden) → check-out (1 tap + optional note) → celebration (subtle) → tomorrow preview. Never more than 2 taps from app-open to practice-start.

---

## Sources (selection)
- thetahealing.com: Reaching the 7th Plane, 7 Planes Meditation, Where Do I Go?, The All That Is, Daily Meditations and Downloads, Manifesting & Abundance course, Digging for Beliefs
- Practitioner: Dawn Maree (YouTube roadmap), Dr. Lisa Cooney (digging), The Goddess Rooms (Dig Deeper), healingwiththeta.com (muscle testing), Quantamind (M&A), Insight Timer (morning downloads meditation)
- Hebrew: rmalul.com, maagar-hashitot.org, reidman.co.il, nogabenisrael.com, Haaretz wellbeing, Walla health
- App mechanics: themindfulnessapp.com 2025 comparison, Healthline Calm-vs-Headspace, PMC mood check-in study, Insight Timer help center, wakingup.com, Five Minute Journal
- Gamification: Yu-kai Chou streak design, StriveCloud Headspace analysis, trophy.so, plotline.so
- RTL/UX: Material Design bidirectionality, UX Collective RTL guide, MDN Web Speech API, Readium speech support tables, Wanderlust 21-day challenge structure
