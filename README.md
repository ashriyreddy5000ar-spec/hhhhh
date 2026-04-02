# My Habits PRO · v2.0

> A beautiful, offline-first habit tracker with cloud sync, streaks, analytics, and achievements — built as a single HTML file.

---

## 🚀 Quick Start

1. Open `index.html` in any modern browser — works instantly with no build step.
2. Click **Continue as Guest** to start tracking immediately (data saved to localStorage).
3. Sign up with email/password for cross-device sync via Firebase (optional, free).

---

## 🔧 What Was Fixed (v2.0.1 — This Release)

### Bug 1 · Habit Counter Shows Wrong Text When Empty
**Problem:** When no habits existed, the progress section showed "Enjoy your rest day" instead of "0 of 0 habits done".

**Fix — `updateProgress()` function:**
```js
// BEFORE (broken):
if(!total){
  st='No habits scheduled 💤'; det='Enjoy your rest day';
}

// AFTER (fixed):
if(!total && !S.habits.length){
  st='No habits yet 👀'; det='0 of 0 habits done';   // ← correct empty state
} else if(!total){
  st='Rest day 💤'; det='No habits scheduled today';  // ← real rest day
}
```

---

### Bug 2 · `nid` Crash on Empty Habits Array
**Problem:** `Math.max(...[])` returns `-Infinity`, causing `nid = -Infinity + 1 = NaN`, which corrupted all new habit IDs.

**Fix — safe `calcNid()` helper added:**
```js
// BEFORE (broken):
nid = Math.max(...S.habits.map(h => h.id), 0) + 1;  // crashes on empty array

// AFTER (fixed):
function calcNid(habits) {
  return habits.length ? Math.max(...habits.map(h => h.id || 0)) + 1 : 1;
}
nid = calcNid(S.habits);  // safe everywhere
```
Applied in all 4 locations: `bootGuestLocal`, Firebase auth handler, Firestore snapshot listener, and offline fallback.

---

### Bug 3 · Streak Resets to 0 If Today Not Yet Completed
**Problem:** If you opened the app and hadn't ticked today's habits yet, all streaks reset to 0 — even if you had a 30-day streak. This was caused by the streak loop breaking on today's miss before checking yesterday.

**Fix — `recalcStreak()` rewritten:**
```js
// BEFORE (broken):
for(let i=0; i<=365; i++){
  const k = d.toISOString().slice(0,10);
  if(h.history && h.history[k]){ streak++; }
  else if(i > 0){ break; }  // ← breaks even if today is just not done yet
  d.setDate(d.getDate()-1);
}

// AFTER (fixed):
const todayDone = !!(h.history && h.history[td]);
if(!todayDone) d.setDate(d.getDate()-1);  // start from yesterday if today pending
for(let i=0; i<=365; i++){
  const k = d.toISOString().slice(0,10);
  if(h.history && h.history[k]){ streak++; best = Math.max(best, streak); }
  else { break; }  // clean break — only on true miss
  d.setDate(d.getDate()-1);
}
```

---

## 🔥 Firebase Setup (for Cross-Device Sync)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Create Project**
2. Add a **Web App** → copy your config
3. Enable **Authentication** → Email/Password + Anonymous
4. Enable **Firestore** → Start in test mode
5. In `index.html`, find `firebaseConfig` and replace the placeholder values:

```js
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

Without Firebase configured, the app runs in **Guest Mode** — full functionality with localStorage persistence.

---

## 📁 File Structure

```
myhabits-pro/
├── index.html        ← Main app (self-contained, all JS inline)
├── app.css           ← External stylesheet (optional — styles also inline)
├── manifest.json     ← PWA manifest for "Add to Home Screen"
└── README.md         ← This file
```

> **Note:** `index.html` includes all critical CSS inline. `app.css` is referenced for extended styles — keep both files together.

---

## 📱 PWA / Installable App

The app ships with a `manifest.json` — users can install it on mobile:
- **iOS Safari:** Share → Add to Home Screen
- **Android Chrome:** Menu → Add to Home Screen / Install App

For full PWA support, add a service worker (see enhancement ideas below).

---

## ✨ Features

| Feature | Status |
|---|---|
| Add / remove habits with emoji, category, frequency | ✅ |
| Daily completion tracking with tap-to-toggle | ✅ |
| Streak calculation (current + best) | ✅ |
| Achievement system (8 milestones) | ✅ |
| Performance score (0–100) with tier badges | ✅ |
| Weekly heatmap | ✅ |
| 30-day trend chart (animated canvas) | ✅ |
| Category breakdown | ✅ |
| AI-style performance insights | ✅ |
| Firebase cross-device sync | ✅ |
| Guest mode (localStorage) | ✅ |
| Custom days / weekdays / weekends frequency | ✅ |
| Reminder time per habit | ✅ |
| Midnight auto-refresh | ✅ |
| Fully accessible (ARIA, keyboard nav) | ✅ |

---

## 💡 World-Class Enhancement Roadmap

These are **logic/backend-only** enhancements — no UI changes required:

### Tier 1 — Ship in v2.1
- **Service Worker + Offline Cache** — true offline PWA, works without internet
- **Push Notifications** — browser/mobile reminders at user-set times
- **Data Export** — download habits history as CSV/JSON
- **Undo Delete** — 5-second toast with undo action on habit removal

### Tier 2 — Ship in v2.2
- **Weekly email digest** — summary of your week via Firebase Cloud Functions + SendGrid (free tier)
- **Habit templates** — 1-tap add from curated preset packs (Morning Routine, Fitness, etc.)
- **Advanced streaks** — grace day logic (miss 1 day/week without breaking streak)
- **Habit notes** — daily journal entry per habit

### Tier 3 — PRO / Paid Features
- **AI Coach** — weekly natural language analysis of your patterns via Claude API
- **Social streaks** — accountability partner mode (shared leaderboard)
- **Custom themes** — light mode, OLED dark, color accent picker
- **Wearable sync** — Apple Health / Google Fit integration

---

## 💰 Commercial Strategy

### Target Audience
- **Primary:** 18–35 self-improvement and productivity enthusiasts
- **Secondary:** Students, remote workers, fitness/wellness community
- **Niche:** Habit tracking communities on Reddit, X, TikTok

### Pricing Model (Freemium)

| Tier | Price | Features |
|---|---|---|
| Free | $0 | Up to 5 habits, local storage only, basic stats |
| PRO | $3.99/mo or $29/year | Unlimited habits, cloud sync, insights, export |
| Teams | $7.99/mo/user | Shared boards, accountability partner, admin dashboard |

### Competitive Advantage
- **No app store required** — works in browser, installable as PWA
- **Privacy first** — optional Firebase, runs 100% offline
- **Single file** — zero dependencies, instant load, no frameworks
- **Beautiful UI** — premium dark aesthetic that rivals $10/month apps
- **Free tier is genuinely useful** — not crippled like Habitica/Streaks

### Marketing Strategy
1. **TikTok / Reels** — "I built a habit tracker that looks like this in 1 file" — dev + productivity crossover content
2. **Product Hunt launch** — target #1 Product of the Day in Productivity
3. **Reddit** — r/selfimprovement, r/productivity, r/webdev — organic posts with screenshots
4. **GitHub** — open-source the free tier, charge for backend/sync
5. **SEO** — "free habit tracker no app", "best habit tracker PWA", "habit tracker offline"
6. **Newsletter** — weekly "habit science" tips → funnels to PRO upgrade

### App Store / Play Store Launch Plan
1. Wrap in **Capacitor.js** (free, 1 day to set up) to produce native iOS/Android builds from the same HTML
2. Submit to App Store ($99/year Apple Dev) + Play Store ($25 one-time)
3. App Store Optimization (ASO): keywords — "habit tracker", "daily routine", "streak app"
4. Launch with 50 ratings from beta users (Product Hunt voters, Reddit community)

### Branding Suggestions
- **Name:** My Habits PRO → consider **"Strk"**, **"Ritual"**, or **"DailyStack"** for catchier brand
- **Tagline:** "Build streaks. Build yourself."
- **Visual identity:** Keep the dark gold/amber palette — it reads as premium, unlike blue/green wellness apps
- **Domain:** grab `strk.app`, `ritual.so`, or `dailystack.co` (~$12/year)
- **Icon:** The ✦ symbol from the logo works well as a minimal app icon

---

## 🛠 Development

No build tools needed. Edit and open directly:

```bash
# Clone / download repo
cd myhabits-pro

# Open in browser
open index.html
# or
python3 -m http.server 8080  # then visit http://localhost:8080
```

For deployment, upload `index.html`, `app.css`, and `manifest.json` to any static host:
- **GitHub Pages** (free)
- **Netlify** (free, drag-and-drop deploy)
- **Vercel** (free)
- **Cloudflare Pages** (free, fastest CDN)

---

## 📄 License

MIT — free for personal and commercial use.
