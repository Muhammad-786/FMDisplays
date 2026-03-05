# FMDisplays — Mosque Digital Signage System
## Complete Setup and Configuration Guide — v4.0

---

## Table of Contents

1. [Overview](#overview)
2. [System Requirements](#system-requirements)
3. [Repository Structure](#repository-structure)
4. [Initial Setup](#initial-setup)
5. [Firebase Configuration](#firebase-configuration)
6. [Admin Panel Guide](#admin-panel-guide)
7. [Display Configuration](#display-configuration)
8. [Android App (NoorBox)](#android-app-noorbox)
9. [GitHub Actions CI/CD](#github-actions-cicd)
10. [API Configuration](#api-configuration)
11. [Troubleshooting](#troubleshooting)
12. [Security Best Practices](#security-best-practices)
13. [Advanced Features](#advanced-features)
14. [Version History](#version-history)

---

## Overview

FMDisplays is a real-time, cloud-synchronised digital signage system for mosques. It runs on web browsers, smart TVs, and dedicated Android devices (Fire Stick / Android TV via the NoorBox APK).

### Components

| Component | File / Repo | Purpose |
|---|---|---|
| **Display** | `index.html` | TV screen — prayer times, Jummah schedule, slideshow |
| **Admin panel** | `admin.html` | Secure management panel for all settings |
| **Android app** | `NoorBoxApp` repo | Kotlin/WebView APK for Fire Stick and Android TV |
| **Web CI/CD** | `FMDisplays/.github/workflows/deploy.yml` | Validates HTML + deploys to GitHub Pages |
| **Android CI/CD** | `NoorBoxApp/.github/workflows/build.yml` | Builds APK + creates GitHub Release |

### Key Features

- Real-time prayer times from Aladhan API or uploaded annual CSV timetable
- Automatic Jummah countdown on Fridays
- Customisable slideshow with image uploads
- 5 colour themes (Gold, Emerald, Royal, Rose, **Custom RGB**)
- 4 layout modes (Classic, Media Focus, Fullscreen, Vertical)
- Firebase Firestore cloud synchronisation — multi-tenant (one project, many mosques)
- Mobile-responsive admin panel (iOS/Android friendly, native time pickers)
- First-time setup wizard for new mosques
- Bilingual display — English + Urdu prayer names
- 4-week Jamaat schedule (different times per week of the month)
- Annual timetable CSV upload (prayer begin times + optional Jamaat columns)
- Configurable countdown-to-Jamaat duration (1–60 minutes)
- **Multi-screen sync** — all TVs always show the same slide simultaneously (epoch-based, no Firestore writes)
- **24/7 hardened listeners** — change detection, debounce, generation counter, auto-restart
- **Fire TV optimised** — desktop UA, fullscreen, keep-screen-on, no GPU crash
- **Built-in slides** — phone-silence and donate reminders (editable English + Urdu text)

---

## System Requirements

### Web Display
- Modern web browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Internet connection for Firebase and Aladhan API
- 1920×1080 display (recommended)

### Android / Fire Stick
- Fire TV Stick Gen 1 or 2, or any Android TV device (Android 5.1+ / API 22+)
- Internet connection
- NoorBox APK (built automatically by GitHub Actions; download from Releases)

---

## Repository Structure

Two separate Git repositories:

| Repo | URL | Contents |
|---|---|---|
| **Web** | `https://github.com/Muhammad-786/FMDisplays.git` | `index.html`, `admin.html`, `instructions.md`, CI/CD workflow |
| **Android** | `https://github.com/Muhammad-786/NoorBoxApp.git` | Kotlin project, bundled `index.html`, CI/CD workflow |

### Web repo layout
```
FMDisplays/
├── index.html
├── admin.html
├── instructions.md
└── .github/
    └── workflows/
        └── deploy.yml
```

### Android repo layout
```
NoorBoxApp/
├── app/
│   └── src/main/
│       ├── kotlin/com/shaheer/noorbox/
│       │   ├── MainActivity.kt
│       │   ├── BootReceiver.kt
│       │   └── NetworkReceiver.kt
│       ├── assets/
│       │   └── index.html        <- bundled copy of display
│       └── res/values/themes.xml
└── .github/workflows/build.yml
```

---

## Initial Setup

### Step 1: Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/) → Add Project
2. **Enable Firestore Database** — Build → Firestore Database → Create database (production mode)
3. **Enable Authentication** — Build → Authentication → Email/Password → Enable
4. **Create admin user** — Authentication → Add user → enter email + password
5. **Enable Storage** (for image uploads) — Build → Storage → Get started

### Step 2: Paste Firebase Config

1. Firebase Console → Project Settings → Your apps → Web (`</>`)
2. Register the app, copy the `firebaseConfig` object
3. Paste it into both `admin.html` and `index.html`, replacing the existing config block

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_ID",
    appId: "YOUR_APP_ID"
};
```

### Step 3: First-Time Setup Wizard

1. Open `admin.html` in a browser
2. Log in with the email/password you created
3. The **Setup Wizard** launches automatically for new accounts:
   - **Step 1 — Mosque:** English name, town/city, Urdu/Arabic name (optional)
   - **Step 2 — Prayer API:** City, country, calculation method, juristic school
   - **Step 3 — Jamaat:** Default congregation times (applied to all 4 weeks)
   - **Step 4 — Jumu'ah:** Speech, Khutbah, and Jamaat times
4. Click **Finish & Save** — the display is live immediately

---

## Firebase Configuration

### Firestore Data Structure

Each mosque account is stored under `tenants/{uid}/` (multi-tenant — one Firebase project, many mosques).

```
tenants/{uid}/                     <- per-mosque document
    mosqueName:       "string"
    mosqueLocation:   "string"
    mosqueNameAr:     "string"     <- Urdu / Arabic name (optional)
    theme:            "gold"       <- gold | emerald | royal | rose | custom
    layout:           "classic"    <- classic | media-focus | fullscreen | vertical
    customColor:      "#7c3aed"    <- used when theme = "custom"
    countdownMinutes: 15           <- minutes before Jamaat for countdown overlay
    prayerTimesMode:  "auto"       <- auto | upload
    jamaatTimesMode:  "manual"     <- manual | upload
    prayerConfig:     { city, country, method, school }
    weeklyPrayers:    [            <- 4-element array (weeks 1-4)
        { fajr:{h,m}, dhuhr:{h,m}, asr:{h,m}, maghrib:{h,m}, isha:{h,m} },
        { ... },
        { ... },
        { ... }
    ]
    annualTimetable:  [ ... ]      <- CSV rows (if prayerTimesMode = "upload")
    jummah:           {
        startDisplayHour, startDisplayMinute,
        speechHour, speechMinute,
        khutbahHour, khutbahMinute,
        jamaatHour, jamaatMinute
    }

tenants/{uid}/slides/              <- slides subcollection
    {slideId}/
        url:      "https://..." | "data:image/..."
        visible:  true
        order:    0
        builtin:  true             <- only on built-in slides
        subtype:  "silent" | "donate"
        titleEn, titleUr, bodyEn, bodyUr
```

---

## Admin Panel Guide

Open `admin.html` in any browser — works on desktop and mobile phones.

### Login

Enter your Firebase email and password. The display URL for your TV screens is shown at the top after login.

### Mosque Info Bar

At the top of the settings view, set:
- **Mosque Name (English)** — shown on the display header
- **Town / Location** — shown under the mosque name
- **Mosque Name in Urdu** (optional) — shown in Arabic script on the display

---

### Timings Tab

**Jumu'ah Schedule**  
Four time-picker inputs: Speech Time, Khutbah Starts, Jumu'ah Jamaat, Countdown Start Time.

**Countdown Before Jamaat**  
How many minutes before each Jamaat the full-screen countdown overlay appears (range: 1–60, default: 15).

**Daily Prayer Jamaat Mode**

| Mode | How it works |
|---|---|
| **Manual (4-Week Schedule)** | Set times per week in the W1/W2/W3/W4 tabs |
| **From Annual Timetable** | Uses Jamaat columns from an uploaded CSV |

**4-Week Manual Schedule**

Each week tab (W1–W4) has 5 native time-picker inputs (Fajr → Isha). The green dot marks the current active week.

- Week 1 — days 1–7
- Week 2 — days 8–14
- Week 3 — days 15–21
- Week 4 — days 22–end

---

### Slides Tab

**Upload image:** Choose file → Upload  
(auto-compressed to 800px wide, 60% JPEG quality)

**Add via URL:** Paste any image URL → Add

**Manage slides:**
- Eye — toggle visibility
- Up/Down arrows — reorder
- Pencil — edit built-in slide text (English + Urdu title and body)
- Trash — delete permanently

**Built-in slides** (seeded automatically on first load):
- *Mobile Phones* — please switch to silent
- *Donate Generously* — charity reminder

Both are editable; the English and Urdu title/body can be changed via the pencil icon.

---

### Appearance Tab

**Colour Themes:** Gold Classic, Emerald Oasis, Royal Night, Rose Dawn, **Custom Color**  
For Custom Color a colour picker appears — choose any hex; the full dark theme is auto-generated and the admin UI recolours live.

**Layout Modes:**
1. Classic — 40/60 split
2. Media Focus — 25/75 split
3. Fullscreen — media fills screen, floating times overlay
4. Vertical — stacked layout (portrait screens)

---

### API Settings Tab

**Prayer Times Mode:**

| Mode | Description |
|---|---|
| **Automatic** | Fetches begin times from Aladhan API using city/country |
| **Upload Annual Timetable** | CSV with one row per date; exact begin times used |

**Automatic mode:** Set City, Country, Calculation Method, Asr Juristic Method, then click **Load Timetable** to preview the current month with W1–W4 week badges.

**CSV Upload format** (Download template button provided):

| Required columns | Optional Jamaat columns |
|---|---|
| Date, Fajr Begin, Sunrise, Dhuhr Begin, Asr Begin, Maghrib Begin, Isha Begin | Fajr Jamaat, Dhuhr Jamaat, Asr Jamaat, Maghrib Jamaat, Isha Jamaat |

If Jamaat columns are present the display uses them automatically; otherwise the 4-week manual schedule applies.

---

### Saving Changes

Click **Save Changes**. All connected screens update in real time — no refresh needed.

---

## Display Configuration

### Opening the Display

- **File directly:** Open `index.html` in a browser
- **GitHub Pages:** `https://muhammad-786.github.io/FMDisplays/index.html`
- **Chrome kiosk mode:** `chrome --kiosk --app=https://muhammad-786.github.io/FMDisplays/index.html`

### Display Elements

#### Header
Mosque name, location, Urdu name, real-time clock and date.

#### Prayer Table
Prayer names in English + Urdu. Begins times from API or CSV; Jamaat from 4-week schedule or CSV. Next prayer row highlighted.

#### Jummah Card
Only shown on Fridays. Displays Speech, Khutbah, and Jamaat times.

#### Slideshow
- Cycles every 8 seconds
- **Epoch-based multi-screen sync** — all TVs independently compute the same slide index from the UTC wall clock; every screen is always in step with no Firestore writes
- Built-in silent-mode and donate slides
- Ken Burns zoom effect (disabled on Fire TV to prevent GPU OOM crash)
- Progress bar shows current slide position

#### Countdown Overlays

**Jamaat countdown** (minutes configured in admin):  
Full-screen overlay with prayer name in English + Urdu and a MM:SS timer. Disappears after Jamaat time.

**Silent-mode reminder** (5 minutes after each Jamaat):  
Full-screen phone-silence prompt, shown for 5 minutes.

#### 24/7 Real-Time Listener Hardening

- **Change detection** — settings/slides only re-applied if data actually changed
- **300ms / 800ms debounce** — rapid Firestore updates batched before processing
- **Generation counter** — stale slideshow initialisations safely discarded
- **Pending timer cleanup** — no timer leaks on rapid setting changes
- **try/catch on setInterval** — slideshow auto-restarts within 2 s after any error
- **setInterval(250ms)** instead of requestAnimationFrame (saves CPU on Fire Stick)
- **10-minute server re-fetch** — bypasses WebSocket cache if TV woke from sleep

---

## Android App (NoorBox)

### Overview

NoorBox is a fullscreen Kotlin/WebView wrapper that loads the mosque display on Fire Stick (Gen 1/2) and Android TV devices.

### Features
- Loads `index.html` from bundled assets (offline-capable for cached Firestore data)
- Desktop user-agent string → display renders in TV layout
- `FLAG_KEEP_SCREEN_ON` — screen never sleeps
- Immersive fullscreen (status bar + navigation bar hidden)
- JS cursor-dot injection on `onPageFinished`
- 2-minute Firebase heartbeat ping
- 10-minute cache clear to force fresh data
- `NetworkReceiver` — reloads page when connectivity restores
- `BootReceiver` — auto-launches on device boot

### Installing the APK

1. Go to **NoorBoxApp → GitHub Actions** → latest successful run → download `NoorBox-debug.apk`  
   **or** GitHub → Releases → grab the latest release APK
2. Sideload onto Fire Stick via ADB or a file-manager app

### Building Locally
```bash
cd NoorBoxApp
./gradlew assembleDebug
# output: app/build/outputs/apk/debug/app-debug.apk
```

---

## GitHub Actions CI/CD

### Web Repo — `deploy.yml`

Triggers on every push to `master`:
1. Validates that `index.html` and `admin.html` exist
2. Deploys the repo to GitHub Pages (`gh-pages` branch)

**One-time setup:**  
GitHub repo → Settings → Pages → Source → `gh-pages` branch  
GitHub repo → Settings → Actions → General → Workflow permissions → Read and write

### Android Repo — `build.yml`

Triggers on every push to `master`:
1. Runs `gradle wrapper --gradle-version=8.2 --distribution-type=bin` (regenerates `gradle-wrapper.jar`, fixing ClassNotFoundException)
2. Runs `./gradlew assembleDebug`
3. Uploads APK as a workflow artifact
4. Creates a GitHub Release with the APK attached

---

## API Configuration

### Aladhan API

**Endpoint:**
```
https://api.aladhan.com/v1/timingsByCity?city={city}&country={country}&method={method}&school={school}
```

**Update frequency:** On load, after admin saves API settings, every 60 minutes.

**Calculation methods:**

| Value | Method |
|---|---|
| 3 | Muslim World League (recommended for UK/Europe) |
| 2 | ISNA (North America) |
| 5 | Egyptian General Authority |
| 4 | Umm Al-Qura, Makkah |
| 1 | University of Islamic Sciences, Karachi |
| 13 | Turkey (Diyanet) |

---

## Troubleshooting

### Admin Panel

| Problem | Solution |
|---|---|
| Can't log in | Verify credentials in Firebase → Authentication |
| Changes not saving | Check console (F12); verify internet connection |
| Slides not uploading | File likely too large for Firestore (1 MB limit); use an external URL instead |
| Mobile layout issues | Hard-refresh; confirm you're on the latest version |

### Display

| Problem | Solution |
|---|---|
| Prayer times not showing | Check city/country in API Settings; verify internet |
| Wrong Asr time | Set juristic method to Hanafi in API Settings |
| Jummah card not showing | Only appears on Fridays; check system date |
| Slideshow not moving | Add at least one visible slide in admin |
| Stale data after TV wake | Wait up to 10 min for auto re-fetch, or hard-refresh |
| Screens out of sync | Ensure all TVs have accurate NTP system clocks |
| App crash on Fire Stick | Download latest APK from GitHub Actions and reinstall |

### Firebase

| Problem | Solution |
|---|---|
| Permission denied | Check Firestore rules allow authenticated writes |
| Data not syncing | Verify Firestore is enabled on Spark or Blaze plan |
| Quota exceeded | Check Firebase Console → Usage |

---

## Security Best Practices

### Firebase Credentials

Firebase API keys in the HTML files are client-side by design — Firebase security is enforced by Firestore rules and Authentication, not key secrecy. For additional protection:

**Option 1 — Domain restriction**  
Firebase Console → Project Settings → API key restrictions → restrict to your GitHub Pages domain.

**Option 2 — Firebase Hosting**
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
firebase deploy --only hosting
```

**Option 3 — App Check**  
Enable Firebase App Check to block unauthorised apps from accessing Firestore.

### Admin Access

- Use a strong password (16+ characters)
- Use a dedicated admin email account
- Avoid accessing the admin panel on public Wi-Fi
- Periodically review Firebase Authentication → Users for unexpected accounts

---

## Advanced Features

### Multi-Screen Synchronised Slideshow

All screens compute the current slide index deterministically:
```javascript
epochIdx = Math.floor(Date.now() / SLIDE_DURATION) % numSlides
```
Every device reads the same UTC epoch → same index → same transition moment.  
Works with unlimited screens, no coordination server and no Firestore writes.

### Annual Timetable CSV

For mosques using a fixed annual prayer timetable:

1. Download the CSV template from API Settings → Upload Annual Timetable tab
2. Fill in dates and prayer times (one row per day)
3. Optionally add Jamaat columns
4. Upload in the Prayer API tab and set mode to **Upload Annual Timetable**

Today's date is matched against the CSV automatically.

### Kiosk Mode

**Windows / Chrome:**
```
chrome.exe --kiosk --app=https://muhammad-786.github.io/FMDisplays/index.html
```
Add a shortcut to `shell:startup` for auto-launch on boot.

**Raspberry Pi:**
```bash
# /etc/xdg/lxsession/LXDE-pi/autostart
@xset s off
@xset -dpms
@xset s noblank
@chromium-browser --noerrdialogs --disable-infobars --kiosk https://muhammad-786.github.io/FMDisplays/index.html
@unclutter -idle 0
```

### Custom CSS Branding

Edit the CSS variables at the top of `index.html` for deeper colour customisation:
```css
:root {
    --gold-primary: #d4af37;
    --gold-light:   #f3e5ab;
    --gold-dark:    #aa8c2c;
}
```

---

## Quick Reference

### Common Admin Tasks

| Task | Steps |
|---|---|
| Change all Jamaat times | Timings → W1/W2/W3/W4 tabs → edit → Save |
| Change Jumu'ah times | Timings → top section → 4 time fields → Save |
| Change countdown duration | Timings → Countdown Before Jamaat → Save |
| Add a slide | Slides → Upload file or paste URL → Add |
| Edit built-in slide text | Slides → pencil icon |
| Change theme | Appearance → select → Save |
| Set custom brand colour | Appearance → Custom Color → pick → Save |
| Update mosque name / location | Mosque Info Bar at top of settings → Save |
| Update prayer API location | API Settings → edit city/country → Save |
| Preview monthly timetable | API Settings → Load Timetable |
| Upload annual timetable | API Settings → Upload Annual Timetable tab → drag & drop CSV |
| Download latest APK | NoorBoxApp GitHub → Actions or Releases |

### Keyboard Shortcuts

| Action | Shortcut |
|---|---|
| Fullscreen (Windows/Linux) | F11 |
| Fullscreen (Mac) | Cmd + Ctrl + F |
| Hard refresh | Ctrl+F5 / Cmd+Shift+R |
| Browser console | F12 |

---

## Version History

**v4.0 — March 2026**
- Multi-screen slideshow sync (epoch-based — all TVs always show the same slide)
- Mobile-responsive admin panel (full-width on phones, natural scroll)
- All HH/MM number-pair inputs replaced with native time pickers
- Touch-friendly: 44 px min tap targets, no iOS auto-zoom, proper flex layout
- Setup wizard mobile fixes: scrollable wizard, 3-column time grid, wrapping rows
- Fixed mosque-name left-gap bug in mobile admin (align-items: stretch)
- GitHub Actions deploy workflow for web repo (validates + deploys to GitHub Pages)
- Android app: AppCompat theme fix, allowUniversalAccessFromFileURLs, body transition disabled (prevents GPU OOM on Fire Stick)
- Jamat times now update in real time on every settings snapshot (no stale display)
- Jummah card hides immediately on non-Fridays at startup
- Removed firestore.rules file (rules managed directly in Firebase Console)

**v3.0 — February 2026**
- Bilingual English + Urdu prayer names with Noto Nastaliq Urdu font
- 4-week Jamaat schedule with week tabs in admin
- Annual timetable CSV upload (prayer begin times + optional Jamaat columns)
- Configurable countdown-to-Jamaat duration (1–60 min)
- Custom RGB colour theme picker
- 24/7 listener hardening (change detection, debounce, generation counter, timer cleanup)
- Auto-sync keep-alive (10-min server re-fetch to survive TV sleep)
- Monthly prayer timetable preview in admin
- First-time setup wizard
- Android NoorBox APK with Fire TV optimisations
- GitHub Actions CI/CD for Android APK builds
- Multi-tenant Firestore schema (tenants/{uid}/)
- Built-in slides (phone silence + donate, editable English/Urdu text)

**v2.0 — February 2026**
- Fixed duplicate school value
- Removed leftover Jummah DOM references
- Improved error handling

**v1.0 — January 2026**
- Initial release

---

*FMDisplays — built for mosques, runs 24/7.*
