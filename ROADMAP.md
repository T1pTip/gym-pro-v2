# 🛣️ ROADMAP — GYM PRO v2

_Live: https://t1ptip.github.io/gym-pro-v2 | Repo: T1pTip/gym-pro-v2_
_מסמך זה נטען אוטומטית בכל סשן Claude לפי `protocols.md` v4.3 → SESSION START CHECKLIST._
_עודכן: 2026-04-26 (rollback Voice Coach + הסרת features מ-roadmap)_

---


## 📋 סטטוס שדרוגים מתוכננים

| # | פיצ'ר | פאזה | סטטוס | זמן | $/חודש | סיכון |
|---|---|---|---|---|---|---|
| - | (אין features פעילים) | - | - | - | - | - |

**מצב נוכחי**: ה-roadmap נוקה אחרי Sprint 1 (Voice Coach) שהסתיים ב-rollback. כל ה-features שהיו ב-roadmap נדחו / בוטלו עם תיעוד ב-Decision Log למטה.

**Features שעלולים לחזור בעתיד** מתועדים ב-`Backlog` בסוף הקובץ. כדי להוסיף feature חדש — להשתמש ב-Workflow למטה וליצור רישום חדש בטבלה.

---

## 📌 Decision Log

**2026-04-25** — Roadmap created.
- Selected: Voice Coach (P1) + Dashboard (P1) + Push Notifications (P2-deferred).
- Rejected for now:
  - **AI Plan Generator** (Claude Haiku) — לא תואם את שלב המוצר הנוכחי. חיוני רק אם המטרה היא צמיחת user base אגרסיבית. דורש backend (CF Worker) + privacy/safety overhead.
  - **Cloud Sync** (Firestore) — דורש privacy compliance (חוק הגנת הפרטיות הישראלי) + email upgrade flow. כדאי רק כשיש 1000+ משתמשים פעילים.

---

**2026-04-26** — Roadmap cleared after Voice Coach rollback.

### ❌ Voice Coach — CANCELLED (was P1)
**שורש הביטול**: speechSynthesis לא משמיע אודיו ב-Android PWA installed mode (Trusted Web Activity / WebAPK). Browser יורה start/end events אבל אין output בפועל. ה-feature לא ניתן לbדיקה client-side ולא ניתן לתיקון בJavaScript.

**מסקנה מתודולוגית**: Risk classification היה שגוי (סווג Risk 1, היה Risk 3). מתודולוגיה מעודכנת ב-LEARNINGS L65-L66 — features שתלויים ב-OS-level APIs (audio, notifications, sensors) ב-PWA installed דורשים Risk 3 כברירת מחדל.

**Bug archived**: BUG-033 (Hebrew silent drop), BUG-034 (PWA installed Android silent speechSynthesis output).

**Sprint duration**: 5 deployments (v9 → v13) → rollback ל-v3.1 (gym-pro-v14 cache).

### ❌ Dashboard / Heat-Map — REMOVED (was P1)
**סיבת הסרה**: Out of scope לשלב הנוכחי. אין מספיק data משמעותי לdashboard בלי **Set Logging** (היה P2 ברעיונות, לא נבנה). בלי volume/sets, ה-dashboard מציג רק streak + ימי אימון — שכבר זמין בUI הקיים.

**אם יחזור בעתיד**: דורש קודם Set Logging (משתמש מקליד weight × reps אחרי כל סט). זה דרך ארוכה ולא ברורה ROI שלה כרגע.

### ❌ Push Notifications — REMOVED (was P2 deferred)
**סיבת הסרה**: אותה משפחת בעיות של Voice Coach.
- iOS Safari pre-16.4: לא נתמך כלל
- Android PWA installed: tracking לא אמין (Doze mode, FCM משתנה)
- אין client-side detection אם הודעה הגיעה
- דורש backend חדש (Cloudflare Worker / VAPID) → כ-6-8 שעות setup
- Risk 3 לפי ה-rubric החדש

**אלטרנטיבה אם הצורך יחזור**: in-app reminders ב-localStorage (Risk 0) — "לא היית 3 ימים, רוצה להמשיך?" pop-up בכניסה לאפליקציה.



## 🗑️ Backlog (Considered, Not Active)

רעיונות שהועלו ונדחו / בוטלו. נשמרים פה כדי לא להעלות אותם שוב מאפס, ולפתוח מחדש אם דרישות משתנות.

### Voice Coach
- **Status**: Cancelled 2026-04-26 (5 deployments, rolled back)
- **Why**: Android PWA installed silently drops speechSynthesis output
- **Revisit when**: pre-recorded MP3 audio files become viable (alternative implementation), or iOS becomes primary platform

### Dashboard / Heat-Map
- **Status**: Removed 2026-04-26 (never started)
- **Why**: Out of scope — דורש Set Logging שלא קיים
- **Revisit when**: Set Logging implemented + 30+ ימי data נצברו

### Push Notifications
- **Status**: Removed 2026-04-26 (never started)
- **Why**: Risk 3 (cross-platform divergence + no client detection + backend dependency)
- **Revisit when**: iOS 16.4+ adoption reaches 90%+ AND Android PWA push proven reliable in production data

### AI Plan Generator (Claude Haiku integration)
- **Status**: Rejected 2026-04-25
- **Why**: Parser v3.1 פותר 90% מהצורך. דורש backend + privacy/safety overhead.
- **Revisit when**: pivot ל-user-base growth + monetization, או אם משתמשים מבקשים "צור לי תוכנית" יותר מ-3 פעמים בחודש

### Cloud Sync (Firestore / Supabase)
- **Status**: Rejected 2026-04-25
- **Why**: Privacy compliance overhead (חוק הגנת פרטיות הישראלי) + email upgrade flow + עלות חודשית. localStorage מספיק ל-single-device user.
- **Revisit when**: 1000+ active users, או דרישה מ-Gil לסנכרן בין מכשירים

### Telegram Reminders Bot
- **Status**: Idea (deferred indefinitely)
- **Why**: Push Notifications המסורתי בוטל; Telegram יכול להיות אלטרנטיבה אבל overhead גבוה (משתמש חייב Telegram, צריך bot, צריך scheduling)

### Apple Health / Google Fit Sync
- **Status**: Idea (deferred indefinitely)
- **Why**: PWA limitations — Health Kit מחייב native app, Google Fit API חלקי

### Form Videos / GIFs per Exercise
- **Status**: Idea (deferred indefinitely)
- **Why**: content effort עצום, copyright concerns. ה-📈 history button + image search הקיימים נותנים reasonable proxy.

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
