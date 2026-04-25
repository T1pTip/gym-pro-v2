# 🛣️ ROADMAP — GYM PRO v2

_Live: https://t1ptip.github.io/gym-pro-v2 | Repo: T1pTip/gym-pro-v2_
_מסמך זה נטען אוטומטית בכל סשן Claude לפי `protocols.md` v4.3 → SESSION START CHECKLIST._
_עודכן: 2026-04-25_

---

## 📋 סטטוס שדרוגים מתוכננים

| # | פיצ'ר | פאזה | סטטוס | זמן | $/חודש | סיכון |
|---|---|---|---|---|---|---|
| 1 | 🎙️ Voice Coach | Phase 1 | 🟡 Pending | ~2 שעות | 0 | Risk 1 |
| 2 | 📊 Dashboard | Phase 1 | 🟡 Pending | ~יום | 0 | Risk 1 |
| 3 | 📱 Push Notifications | Phase 2 | ⏸️ Deferred | חצי יום | 0 | Risk 1 |

**סדר עדיפות מומלץ:** 1 → 2 → 3.
- Voice Coach ראשון — quick win עם ROI מקסימלי, אפס תלויות.
- Dashboard שני — דורש storage חדש שיהיה זמין מיד.
- Push דחוי — Voice Coach מכסה 80% מהערך. להפעיל רק אם משתמשים מדווחים שלא מספיק.

---

## 1. 🎙️ Voice Coach — `[Pending — Priority 1]`

### תמצית
הקראת רמזים קוליים בנקודות מפתח: התחלת/סיום rest timer, מעבר תרגיל, סיום אימון. Hands-free workflow — טלפון נשאר בכיס, אוזניות באוזניים.

### Success Metrics (VP Product)
- המשתמש מתאמן עם אוזניות + מסך כבוי ועדיין מקבל את כל המידע על הסט הבא
- הקטנת זמן בין-סטים מ-150-180s (משתמשים שמסתכלים בטלפון) ל-90-100s
- Accessibility: משתמש לקוי-ראייה יכול להשתמש באפליקציה

### Stack
- **Web Speech API** — `SpeechSynthesisUtterance` + `speechSynthesis.speak()`
- 0 dependencies חיצוניות, 0 API costs
- סינתוז מקומי (Carmit ב-iOS, Google TTS ב-Android Chrome, Asaf/Carmit ב-Windows)

### Implementation Plan — 4 trigger points

**A. פונקציה מרכזית `speak(text)`** — חדשה
- localStorage key חדש: `gym_voice` (on/off, default on)
- Cancel pending utterances לפני speak חדש (חשוב!)
- Voice selection: `he-IL` או fallback ל-`carmit`/`asaf`
- ~25 שורות

**B. Toggle ב-dropdownMenu**
- כפתור 🎙️ קול ב-`dropdownMenu` div
- Function `toggleVoice()` שעודכן localStorage
- ~10 שורות

**C. 4 Trigger Points (במקומות הקיימים)**
1. `startTimer(exName)` — בתחילת מנוחה
2. בתוך setInterval כש-`timerRemain === 10` — אזהרה 10 שניות
3. כש-timer מסתיים — קריאה ל-`speakNextSet()`
4. `showSummary(exCount, prCount)` — סיכום קולי

**D. `speakNextSet()`** — חדשה
- מאתרת את התרגיל הבא שלא הושלם (`!done[curDay+'_'+i]`)
- בונה הודעה מ: ex.name + sets + reps + savedW (אם יש) + ex.rpe (אם יש)
- מתחשבת ב-weekVariants (w1/w4) לפי weekModes
- ~25 שורות

### Adversarial Pre-Check (5 attack vectors)
1. **iOS user-gesture requirement** — `speechSynthesis` חסום לפני user gesture ראשון. הקול הראשון יוצמד ל-`doLoad()` או click על "סיימתי" — לא יעבוד autoplay בעת טעינה. PROOF: בדיקה ב-Safari iOS עם speak() בעת DOMContentLoaded → no audio.
2. **`getVoices()` race condition** — חוזר `[]` בטעינה ראשונה. Fix: `speechSynthesis.addEventListener('voiceschanged', ...)` once, או polling.
3. **Cancel race** — אם המשתמש לוחץ "סיימתי" מהר 2 פעמים, ההודעות מצטברות בתור. Fix: `speechSynthesis.cancel()` חובה לפני כל `speak()` חדש.
4. **משתמש משתיק טלפון** — להציג גם ויזואלית את שם התרגיל הבא (timerExercise כבר מציג את הנוכחי, נצטרך להרחיב).
5. **PWA standalone iOS + מסך נעול** — קול לא עובד ברקע. שמירה על מסך דולק עם Wake Lock API — אופציונלי, לא חובה ב-MVP.

### Risk Classification: Risk 1
- שינוי single-file ב-`index.html`
- ~80 שורות JS חדשות
- אין dependencies חדשים, אין שינויי CSP, אין cost impact
- לפי `risk-classifier.md`: declare + verify after change

### Acceptance Criteria (Evidence Standard)
לפני סיווג כ-`done` — חובה evidence ולא claim:
- ✅ Toggle ב-dropdown עובד ושומר ב-localStorage (test: refresh → state persists)
- ✅ Voice קוראת hint בתחילת timer בעברית **וגם** באנגלית
- ✅ Voice קוראת שם תרגיל הבא + sets/reps + weight + RPE בסיום timer
- ✅ Voice קוראת summary בסיום אימון
- ✅ Adversarial: iOS Safari (Carmit), Chrome Android, Chrome Desktop — קוראת בקול הנכון
- ✅ Adversarial: cancel() עובד אם המשתמש לוחץ "סיימתי" 2 פעמים מהר
- ✅ Adversarial: voiceschanged race לא גורם לקריאה במבטא לא נכון
- ✅ Live verification דרך Claude in Chrome: 3+ runs בלי שגיאות

### Future Extensions (לא ב-MVP)
- 🎤 **Voice Commands** — `SpeechRecognition` לפעולות "סיימתי", "דלג", "עוד 30 שניות"
- 🧠 **Coach Personalities** — Drill Sergeant / Calm / Data Geek (אותו מילון בניסוחים שונים)
- 🔔 **Tips per category** — מילון רמזי טכניקה לפי `ex.cat` ("שמור על הכתפיים אחורה" לחזה)

---

## 2. 📊 Dashboard עם Heat-Map שבועי — `[Pending — Priority 2]`

### תמצית
מסך חדש שמציג היסטוריה ויזואלית: heat-map של 90 ימים בסגנון GitHub contributions, גרף נפח 12 שבועות, top 3 PRs של החודש, רצף + 4 stat cards.

### Success Metrics (VP Product)
- **Retention loop** — משתמש פותח את האפליקציה ב-rest day גם ללא תוכנית
- ויזואליזציה של streak מגדילה motivation להמשיך
- "Strava effect" — האפליקציה הופכת ממקום שפותחים בחדר כושר למקום שחוזרים אליו

### Stack
- Canvas 2D (כמו `histCanvas` הקיים — אותם דפוסים)
- localStorage חדש: `gym_workout_log` (מערך של רשומות אימון)
- 0 dependencies חיצוניות

### תלות חיונית — Pre-requisite
**חייב לבוא לפני המסך**: רישום log של כל אימון. נדרש מבנה storage חדש:

```js
gym_workout_log: [
  {
    date: '2026-04-25',     // ISO short
    dayIdx: 0,
    dayName: 'אימון A',
    exercises: 4,            // total exercises completed today
    prs: 1,                  // PRs hit today (count)
    durationSec: 2820,       // from getWorkoutElapsed()
    volume: 12450,           // sum of (sets × reps × kg) for measured exercises
    timestamp: 1714060800000 // epoch ms — for sorting
  }
]
```

נקרא מ-`closeSummary()` או `showSummary()`. אם לא, היסטוריה אבודה. **רק אחרי שהlog פעיל אפשר לבנות את המסך**.

### רכיבי המסך

**A. Heat-Map (90 ימים)**
- Grid: 13 שבועות × 7 ימים על mobile 380px (~14px ריבוע + 2px gap)
- צבעים לפי volume:
  - אפור (`var(--card2)`) = יום ריק
  - אקצנט-עמום (rgba(129,140,248,0.15)) = volume נמוך (<3000)
  - אקצנט (`var(--accent)`) = volume בינוני (3000-8000)
  - אקצנט-זוהר עם hue shift = volume גבוה (>8000)
- Hover/tap על ריבוע = tooltip עם date + exercises + volume (kg)

**B. גרף נפח שבועי**
- Bar chart של 12 שבועות אחרונים
- גובה bar = sum(volume) של השבוע
- Trend line עליון אם volume עולה לאורך זמן

**C. Top 3 PRs של החודש**
- שליפה מ-`gym_prs_<dateStr>` (כבר קיים — נצטרך iteration על 30 ימים אחרונים)
- list עם: שם תרגיל + תאריך + הפרש kg מהקודם (חישוב מ-`gym_weight_history_v2`)

**D. סטטיסטיקות פתיחה (4 cards)**
1. **רצף נוכחי** — `gym_streak_v1.count`
2. **שעות אימון השבוע** — sum(durationSec) של 7 ימים אחרונים, format hh:mm
3. **תרגיל מוביל** — תרגיל שעלה הכי הרבה kg ב-30 ימים אחרונים (delta max-min)
4. **Volume PR** — נפח שבועי הגבוה ביותר אי-פעם

### Implementation Hints
- כפתור 📊 ב-dropdownMenu לפתיחת מסך
- Modal overlay (אותו דפוס כמו histModal הקיים)
- ב-`devicePixelRatio === 2`: scale(2,2) על canvas + width*2/height*2 (כמו histCanvas)
- Volume של תרגיל = (sets * mid-of-reps-range * kg). תרגילי משקל-גוף = 70kg כברירת מחדל, או skip
- `computeVolume(day, dayIdx)` helper חדש — סה"כ volume של יום אימון

### Adversarial Pre-Check (6 attack vectors)
1. **History gap** — משתמשים קיימים ייתקלו ב-empty state. Mitigation: backfill חלקי מתאריכי `gym_weight_history_v2` (אבל ללא volume/duration). UX: empty state עם "התאמן עוד 3 ימים כדי לראות trend".
2. **Mobile width 380px** — heat-map עלול לגלוש. תכנון: 13 cols × 14px + 12 gaps × 2px = 206px ✓ נכנס.
3. **Canvas devicePixelRatio** — אותה בעיה כמו histCanvas. Fix: scale(2,2) + canvas.width = offsetWidth*2.
4. **Volume calc עם תרגילי משקל-גוף** — אם isAb()/calisthenics, אין משקל. Decision: skip את התרגיל בvolume sum (כי לא רלוונטי) ולא לחשיב bodyweight default.
5. **Storage growth** — 365 רשומות × ~150 bytes = ~55KB. בסדר גמור (5MB localStorage limit). Cap: `if(log.length > 365) log.shift()`.
6. **Corrupted log entry** (missing field after schema change) — לא אמור להפיל את המסך. Fix: defensive reads עם `?? 0` ו-`?? ''`.

### Risk Classification: Risk 1
- שינוי single-file ב-`index.html`
- ~250 שורות חדשות (UI + canvas drawing + helpers)
- 1 storage key חדש
- אין שינויי CSP, אין cost impact

### Acceptance Criteria (Evidence Standard)
- ✅ `logWorkout()` נקרא מ-`showSummary` ושומר נכון (test: complete workout → check localStorage)
- ✅ Heat-map מוצג נכון על mobile 380px ו-desktop 1920px
- ✅ Hover/tap על ריבוע מציג tooltip
- ✅ גרף נפח מציג 12 שבועות
- ✅ Top PRs מתעדכן אחרי PR חדש
- ✅ 4 stat cards מחשבים נכון
- ✅ Adversarial: empty state, 1 day, 1 week, 1 year of data — כולם renderים בלי שגיאות
- ✅ Adversarial: corrupted log entry (missing field) לא קורס את המסך

### Future Extensions
- ייצוא CSV של היסטוריה
- השוואה לתקופה קודמת ("השבוע vs שבוע שעבר", "החודש vs חודש שעבר")
- AI-generated insights לפי rules ("המשקל בבנץ עלה 7% החודש 🔥", "5 ימי רצף — שיא חדש!")
- Shareable workout summary card (image)

---

## 3. 📱 Push Notifications — `[Deferred — Priority 3]`

### תמצית
נוטיפיקציה במסך נעול בנקודות מפתח: סיום timer, מעבר תרגיל, סיום אימון. עם Notification Actions ("✓ סיימתי", "+30s") ישירות מהמסך הנעול — בלי לפתוח את האפליקציה.

### למה Deferred
**חופף ל-Voice Coach** ב-80% מהערך. Voice Coach מספק את אותה תקשורת hands-free, ובלי הצורך ב-permission prompt או PWA installation.

**להפעיל רק אם:**
- Voice Coach לא מספק ערך מספיק (משתמשים מדווחים שלא שומעים בחדר כושר רועש או מבכים על מסך נעול)
- 30%+ מהמשתמשים מבקשים את זה ספציפית
- רוצים את ה-Action buttons הייחודיים (סיים תרגיל מבלי לפתוח אפליקציה)

### Stack
- **Web Push API** (`Notification.requestPermission()`, `reg.showNotification()`)
- **Service Worker** (כבר רשום ב-`sw.js`)
- 0 backend — push מקומי, לא מהשרת
- 0 dependencies חיצוניות

### Platform Support Matrix
| פלטפורמה | תמיכה | הערות |
|---|---|---|
| Android Chrome | ✅ מלא | כולל Notification Actions, Vibrate |
| iOS Safari ≥ 16.4 | ⚠️ חלקי | רק PWA installed, לא Safari רגיל |
| iOS Safari רגיל | ❌ אין | חובה התקנה למסך הבית |
| Desktop Chrome/Edge | ✅ מלא | כולל Actions |
| Desktop Firefox | ✅ מלא | כולל Actions |

### Implementation Plan

**A. Permission Flow (חובה contextual)**
- **אסור** לבקש permission ב-cold start (70% rejection rate בתעשייה)
- לבקש רק אחרי שהמשתמש לחץ "התחל אימון" פעם אחת לפחות (יש כבר flag `WDUR_KEY`)
- function `requestPushPermission()` — async, מחזירה boolean
- ~15 שורות

**B. Show Notification מ-trigger points**
- function `notify(title, body, actions)` async
- Default actions: `[{action:'done', title:'✓ סיימתי'}, {action:'add30', title:'+30s'}]`
- `tag: 'gym-pro-rest'` — חשוב לdedupe (replaces previous notification)
- `requireInteraction: true` — נשאר עד שהמשתמש מסיר
- `vibrate: [100, 50, 100]`
- ~25 שורות

**C. Action Handler ב-sw.js**
- `self.addEventListener('notificationclick', ...)`
- focus existing client או openWindow
- postMessage לדף עם `{type: 'notif-action', action: e.action}`
- ~15 שורות

**D. Page Listener**
- `navigator.serviceWorker.addEventListener('message', ...)`
- אם action === 'done' → click על `.check-btn:not(.done-state)` הראשון
- אם action === 'add30' → `addTime(30)`
- ~10 שורות

### Trigger Points (להפעיל push only when document is hidden)
- בדיקה: `document.visibilityState !== 'visible'` לפני כל notify
- 1. סיום timer מנוחה
- 2. מעבר לתרגיל הבא
- 3. סיום אימון (עם summary stats)

### Adversarial Pre-Check (7 attack vectors)
1. **iOS PWA-only limitation** — צריך כיתוב "להתקנה למסך הבית קודם" למשתמשי Safari iOS רגיל. detection: `window.matchMedia('(display-mode: standalone)').matches`.
2. **Permission rejection irreversible** — ברגע שמשתמש דחה (`Notification.permission === 'denied'`), אי אפשר לשאול שוב. UI נדרש: כיתוב "לא הופעל — תפעיל ב-settings → Notifications".
3. **Dedupe via tag** — חיוני מאוד אחרת כל סט = notification חדשה stacked. `tag: 'gym-pro-rest'` חובה.
4. **`document.visibilityState === 'visible'`** — אם הדף פעיל, **לא** לשלוח push (כפילות מיותרת — יש כבר UI). שלח רק כשרקע/מסך נעול.
5. **Wake Lock conflict** — אם נשתמש ב-Wake Lock לשמור על מסך דולק, push פחות חשוב.
6. **Battery impact** — נוטיפיקציה במסך נעול מעירה display. סביר ב-context אימון אבל לציין למשתמש בfirst prompt.
7. **Service Worker not active** — אם SW לא registered/active, `reg.showNotification` יזרוק. Fix: defensive `if (!reg) return;`.

### Risk Classification: Risk 1
- שינוי single-file ב-`index.html` + עדכון `sw.js` (חובה bump CACHE version!)
- ~120 שורות חדשות
- אין שינויי CSP, אין cost impact

### Acceptance Criteria (Evidence Standard)
- ✅ Permission prompt מוצג רק אחרי "התחל אימון" ראשון, לא ב-cold start
- ✅ Notification מופיעה במסך נעול ב-Android
- ✅ Action buttons "סיימתי" ו-"+30s" עובדים מ-notification
- ✅ Notification מתעדכנת (tag dedupe), לא מצטברת
- ✅ visibilityState בדיקה — לא שולחים push כשדף פעיל
- ✅ iOS standalone: אם installed → עובד; אם Safari → fallback graceful עם הסבר
- ✅ Adversarial: דחיית permission לא קורסת את האפליקציה
- ✅ Adversarial: SW לא רשום → לא מפיל עם uncaught error
- ✅ SW cache bump verified (gym-pro-v4 → gym-pro-v5)

### תלות לפני הפיצ'ר
1. **Voice Coach implemented + tested live** — בדיקה אמפירית אם הוא מספיק
2. **User feedback** — אם 30%+ מבקשים push, להפעיל

---

## 📌 Decision Log

**2026-04-25** — Roadmap created.
- Selected: Voice Coach (P1) + Dashboard (P1) + Push Notifications (P2-deferred).
- Rejected for now:
  - **AI Plan Generator** (Claude Haiku) — לא תואם את שלב המוצר הנוכחי. חיוני רק אם המטרה היא צמיחת user base אגרסיבית. דורש backend (CF Worker) + privacy/safety overhead.
  - **Cloud Sync** (Firestore) — דורש privacy compliance (חוק הגנת הפרטיות הישראלי) + email upgrade flow. כדאי רק כשיש 1000+ משתמשים פעילים.

---

## 🔗 References

לפני התחלת עבודה על כל פיצ'ר, יש לקרוא:
- `C:\Users\moran\.claude\protocols.md` v4.3 — 4-Agent Framework + 5 Critical Rules
- `C:\Users\moran\.claude\known-bugs.md` — Bug Museum (BUG-011, BUG-012)
- `C:\Users\moran\.claude\risk-classifier.md` — Risk 0/1/2/3 classification
- `C:\Users\moran\.claude\evidence-standard.md` — לפני סיווג `done`
- `C:\Users\moran\.claude\adversarial-qa.md` — 7 attack vectors
- `LEARNINGS.md` (project root) — לקחים חוצי-סשנים L1-L33

## 🚦 Workflow לפיצ'ר חדש

לפי `protocols.md` 4-Agent Framework:

1. **VP Product** — Define Success Metrics. Reference מהroadmap הזה.
2. **Coding** — Implement against the spec בroadmap.
3. **QA** — 3-layer (Traditional + Adversarial 7 vectors + Evidence Standard).
4. **Retrospective** — Update LEARNINGS.md with `[Global Context]` ועדכן roadmap (סטטוס → ✅ Done).

---

## 📊 Status Tracker

עדכן את הטבלה בראש הקובץ עם סטטוס חדש כשמתחילים/מסיימים:
- 🟡 Pending — בpipeline, לא התחיל
- 🔵 In Progress — בעבודה כרגע (כולל פרטי [OWNER] tag)
- ⏸️ Deferred — דחוי מסיבה כלשהי (ציין סיבה)
- ✅ Done — הושלם, עבר QA + live verified
- ❌ Cancelled — ירד מהroadmap (ציין סיבה)
