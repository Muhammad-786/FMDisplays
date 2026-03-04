# Faizan-E-Madinah Mosque Display System
## Complete Setup and Configuration Guide

---

## 📑 Table of Contents

1. [Overview](#overview)
2. [System Requirements](#system-requirements)
3. [Initial Setup](#initial-setup)
4. [Firebase Configuration](#firebase-configuration)
5. [File Structure](#file-structure)
6. [Admin Panel Guide](#admin-panel-guide)
7. [Display Configuration](#display-configuration)
8. [API Configuration](#api-configuration)
9. [Troubleshooting](#troubleshooting)
10. [Security Best Practices](#security-best-practices)
11. [Advanced Features](#advanced-features)

---

## Overview

The Faizan-E-Madinah Mosque Display System is a real-time, cloud-synchronized digital signage solution for mosques. It consists of two main components:

- **`index.html`** - Public display showing prayer times, Jummah schedule, and slideshow
- **`admin.html`** - Secure control panel for managing all display settings

### Key Features

✅ Real-time prayer times from Aladhan API  
✅ Automatic Jummah countdown on Fridays  
✅ Customizable slideshow with image uploads  
✅ 5 color themes (Gold, Emerald, Royal, Rose, **Custom RGB**)  
✅ 4 layout modes (Classic, Media Focus, Fullscreen, Vertical)  
✅ Firebase cloud synchronization  
✅ Mobile-responsive design  
✅ Automatic prayer countdown overlays  
✅ **Bilingual display — English + Urdu prayer names**  
✅ **4-week Jamaat schedule** (different times per week of month)  
✅ **Configurable countdown-to-Jamaat duration** (1–60 minutes)  
✅ **Custom RGB color theme picker** (pick any brand colour)  
✅ **Auto-sync keep-alive** (force re-fetch every 10 min to survive TV sleep/disconnect)  
✅ **Monthly prayer timetable preview** in admin API Settings  

---

## System Requirements

### Minimum Requirements
- Modern web browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Internet connection for Firebase and API access
- Display device (TV, monitor, tablet, or projected screen)

### Recommended Setup
- **Display Device:** 1920x1080 HD screen or higher
- **Browser:** Chrome or Edge in kiosk mode for unattended operation
- **Network:** Stable broadband connection (5 Mbps minimum)
- **Device:** Raspberry Pi 4, Intel NUC, or dedicated PC

---

## Initial Setup

### Step 1: Firebase Project Setup

1. **Create Firebase Project**
   - Go to [Firebase Console](https://console.firebase.google.com/)
   - Click "Add Project"
   - Enter project name (e.g., "MosqueDisplay")
   - Disable Google Analytics (optional)
   - Click "Create Project"

2. **Enable Firestore Database**
   - In Firebase Console, go to "Build" → "Firestore Database"
   - Click "Create database"
   - Choose production mode
   - Select cloud location closest to your mosque
   - Click "Enable"

3. **Set Firestore Rules**
   ```javascript
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       // Settings document - read by anyone, write by authenticated users only
       match /mosque_data/{document=**} {
         allow read: if true;
         allow write: if request.auth != null;
       }
       
       // Slides collection - read by anyone, write by authenticated users only
       match /slides/{document=**} {
         allow read: if true;
         allow write: if request.auth != null;
       }
     }
   }
   ```

4. **Enable Authentication**
   - Go to "Build" → "Authentication"
   - Click "Get started"
   - Enable "Email/Password" sign-in method
   - Click "Save"

5. **Create Admin User**
   - In Authentication tab, click "Add user"
   - Enter email: `admin@yourmosque.com`
   - Enter a strong password
   - Click "Add user"

### Step 2: Get Firebase Configuration

1. In Firebase Console, go to Project Settings (gear icon)
2. Scroll to "Your apps" section
3. Click the web icon (`</>`) to add a web app
4. Register app with a nickname (e.g., "Display System")
5. Copy the `firebaseConfig` object
6. **Update both files:**

**In `admin.html` (around line 682):**
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

**In `index.html` (around line 1323):**
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

### Step 3: Initial Data Setup

1. **Open `admin.html` in your browser**
2. Log in with the admin credentials you created
3. Firebase will automatically create the initial settings document

---

## Firebase Configuration

### Database Structure

```
mosque_data (collection)
  └── settings (document)
      ├── theme: "gold"               # gold | emerald | royal | rose | custom
      ├── layout: "classic"           # classic | media | fullscreen | vertical
      ├── customColor: "#7c3aed"      # hex colour used when theme = "custom"
      ├── countdownMinutes: 15        # minutes before Jamaat to show countdown overlay
      ├── jummah: { speechHour, speechMinute, khutbahHour, ... }
      ├── prayerConfig: { city, country, method, school }
      └── weeklyPrayers: [            # 4-element array (week 1–4 of month)
            { fajr:{h,m}, dhuhr:{h,m}, asr:{h,m}, maghrib:{h,m}, isha:{h,m} },
            { ... },  // week 2
            { ... },  // week 3
            { ... }   // week 4
          ]

slides (collection)
  ├── slide1 (document)
  │   ├── url: "https://..."
  │   ├── visible: true
  │   └── order: 0
  ├── slide2 (document)
  │   ├── url: "data:image/..."
  │   ├── visible: true
  │   └── order: 1
  └── ...
```

> **Migration note:** The old single `dailyPrayers` field has been replaced by `weeklyPrayers` (array of 4 objects). The display automatically picks the correct week based on the current date (days 1–7 → week 1, 8–14 → week 2, 15–21 → week 3, 22+ → week 4).

---

## Admin Panel Guide

Access the admin panel by opening `admin.html` in any browser.

### Login

- **URL:** `admin.html`
- **Email:** The email you created in Firebase Authentication
- **Password:** Your Firebase auth password

### Timings Tab

Configure all prayer and Jummah times:

#### Jummah Schedule
- **Speech Time:** When pre-Jummah speech/announcements begin
- **Khutbah Starts:** When the actual Khutbah begins
- **Jummah Jamaat:** Main congregation time (displayed on screen)
- **Countdown Start Time:** When to start showing Jummah display on Fridays

#### Daily Prayer Jamaat Times — 4-Week Schedule

Jamaat times can vary across the month (e.g., summer vs winter shifts). The system supports **four independent weekly schedules**:

- **Week 1** — days 1–7 of the month
- **Week 2** — days 8–14
- **Week 3** — days 15–21
- **Week 4** — days 22–end

Each week has its own tab in the Timings section. Click the **W1 / W2 / W3 / W4** tab buttons to switch between weeks and set Fajr, Dhuhr, Asr, Maghrib, and Isha times independently.

The **current week's tab** is marked with a green dot so you always know which schedule is active on the display.

Format: 24-hour (e.g., 06:00 for 6:00 AM, 18:30 for 6:30 PM)

> **Note:** Prayer "begins" times are fetched automatically from the Aladhan API based on your location settings (see API Settings tab).

#### Countdown Duration

The **Countdown to Jamaat Minutes** field (default: 15) controls how many minutes before each Jamaat time the full-screen countdown overlay appears on the display. Valid range: 1–60 minutes.

Set a shorter value (e.g., 5) if you want only a brief countdown, or a longer value (e.g., 20) to give the congregation more notice.

### Slides Tab

Manage slideshow content with two upload methods:

#### Method 1: File Upload
1. Click "Choose File" button
2. Select an image (JPG, PNG, GIF)
3. Click "Upload"
4. Image is automatically compressed and uploaded

**Image Optimization:**
- Maximum width: 800px (auto-resized)
- Quality: 60% compression
- Format: JPEG

#### Method 2: URL Link
1. Enter image URL in the text field
2. Click "Add"
3. External images are loaded directly

#### Slide Management
- **👁️ Eye Icon:** Toggle visibility (hidden slides won't display)
- **↑ Up Arrow:** Move slide earlier in rotation
- **↓ Down Arrow:** Move slide later in rotation
- **🗑️ Trash:** Delete slide permanently

### Appearance Tab

#### Color Themes
Choose from 5 professionally designed color schemes:

1. **Gold Classic** - Traditional gold and dark theme
2. **Emerald Oasis** - Green tones with nature-inspired gradients
3. **Royal Night** - Blue accents with elegant styling
4. **Rose Dawn** - Pink/rose theme with soft gradients
5. **Custom Color** - Pick any RGB hex colour; the system auto-computes a full cohesive dark theme (background, accent light/dark, gradient texture) from your chosen colour

##### Using Custom Color
1. Click the **Custom Color** card in the Appearance tab
2. A colour picker row appears below the theme cards
3. Click the colour swatch / native colour wheel and choose your brand colour
4. The preview bar updates live to show what the theme will look like
5. Click **Save Changes** — the display updates instantly

#### Layout Modes
Optimize display for your screen setup:

1. **Classic** - 40/60 split (schedule on left, media on right)
2. **Media Focus** - 25/75 split (smaller schedule, larger media)
3. **Fullscreen** - Media fills screen with floating schedule overlay
4. **Vertical** - Stacked layout (ideal for portrait displays)

### API Settings Tab

Configure prayer time calculation:

- **City:** Your city name (e.g., "Ravensthorpe", "London")
- **Country:** Country code (e.g., "UK", "US", "CA")
- **Calculation Method:** 
  - Muslim World League (recommended for Europe)
  - ISNA (North America)
  - Egyptian General Authority
  - Umm Al-Qura University, Makkah
  - University of Islamic Sciences, Karachi
  - France (UOIF)
  - Turkey (Diyanet)
- **Asr Juristic Method:**
  - Shafi (Standard)
  - Hanafi

#### Monthly Prayer Timetable Preview

After entering your city/country and clicking **Load Timetable**, a scrollable table for the current month appears below the settings form. It shows:

- Each day's Fajr, Sunrise, Dhuhr, Asr, Maghrib, and Isha **begin** times from the Aladhan API
- A **W1/W2/W3/W4** badge on each row indicating which Jamaat week schedule applies that day
- **Today's row highlighted** in gold for quick reference

This lets you verify that your 4-week Jamaat schedule aligns correctly with the calculated prayer begin times before saving.

### Saving Changes

After making any changes:
1. Click the **"Save Changes"** button at the bottom
2. Wait for the "Saved!" confirmation
3. Changes sync to the display instantly (no refresh needed)

---

## Display Configuration

### Opening the Display

1. Open `index.html` in a web browser
2. For production use, enable **fullscreen mode**:
   - **Windows/Linux:** Press `F11`
   - **Mac:** Press `Cmd + Ctrl + F`
   - **Chrome Kiosk:** Launch with `chrome --kiosk --app=file:///path/to/index.html`

### Display Elements

#### Header
- **Mosque Name:** "Faizan-E-Madinah" (edit in HTML if needed)
- **Location:** "Ravensthorpe" (edit in HTML if needed)
- **Real-time Clock:** Updates every second
- **Date:** Shows current date in long format

#### Daily Prayers Table
- **Prayer Names:** Fajr, Sunrise, Dhuhr, Asr, Maghrib, Isha
- **Begins Column:** Fetched from Aladhan API
- **Jamat Column:** Set manually in admin panel
- **Highlight:** Next upcoming prayer is highlighted

#### Jummah Section
- Only visible on **Fridays**
- Shows Jamaat time prominently
- Automatically updates from admin settings

#### Slideshow
- Cycles through uploaded slides every 8 seconds
- Includes 2 static slides:
  - "Please switch phones to silent mode"
  - "Donate Generously" reminder
- Ken Burns effect (subtle zoom animation)
- Progress bar shows current slide position

#### Countdown Overlays

**Prayer Countdown (configurable minutes before Jamaat):**
- Full-screen black background
- Shows prayer name in English **and Urdu** with countdown timer
- Example: "The Prayer of Asr Is In 14:32" / "نماز عصر"
- Duration controlled by **Countdown to Jamaat Minutes** setting in admin (default: 15 min)
- Automatically hides after Jamaat time

**Silent Mode Reminder (5 minutes after Jamaat):**
- Full-screen display
- Shows phone silence icon
- Displays for 5 minutes after each prayer starts

#### Bilingual Display (English + Urdu)

All prayer names are shown in both English and Urdu script using the **Noto Nastaliq Urdu** font:

| English | Urdu |
|---------|------|
| Fajr | فجر |
| Sunrise | طلوعِ آفتاب |
| Dhuhr | ظہر |
| Asr | عصر |
| Maghrib | مغرب |
| Isha | عشاء |

Column headers (Prayer / Time / Jamaat) and section titles are also bilingual. Urdu text is right-aligned and uses a smaller font to fit neatly alongside the English text.

#### Auto-Sync Keep-Alive (TV Sleep Fix)

On-screen displays connected via HDMI to smart TVs can lose Firebase's real-time WebSocket connection after the TV enters sleep/standby mode. To prevent stale data:

- Every **10 minutes**, the display silently force-fetches settings and slides directly from the Firestore server (bypassing any local cache)
- If settings have changed it re-applies them; if slides have changed it refreshes the slideshow
- No visible interruption to the display — the sync happens in the background
- No configuration required; this runs automatically

---

## API Configuration

### Aladhan API Integration

The system uses the [Aladhan API](https://aladhan.com/prayer-times-api) to fetch accurate prayer times.

#### API Endpoint
```
https://api.aladhan.com/v1/timingsByCity?city={city}&country={country}&method={method}&school={school}
```

#### Update Frequency
- Initial fetch on page load
- Re-fetches every 60 minutes
- Updates when admin changes settings

#### Response Fields Used
- `Fajr` - Dawn prayer
- `Sunrise` - Sun rise (no Jamat)
- `Dhuhr` - Noon prayer
- `Asr` - Afternoon prayer
- `Maghrib` - Sunset prayer
- `Isha` - Night prayer

#### Troubleshooting API Issues

**Problem:** Prayer times not showing
- **Solution 1:** Check internet connection
- **Solution 2:** Verify city/country spelling in admin panel
- **Solution 3:** Open browser console (F12) and check for API errors

**Problem:** Wrong calculation method
- **Solution:** Change "Calculation Method" in API Settings tab

**Problem:** Asr time doesn't match local mosque
- **Solution:** Change "Asr Juristic Method" to Hanafi if needed

---

## Troubleshooting

### Admin Panel Issues

#### Can't Log In
1. Verify email and password in Firebase Console → Authentication
2. Check browser console for errors (F12)
3. Ensure Firebase config is correct in `admin.html`
4. Try resetting password in Firebase Console

#### Changes Not Saving
1. Check internet connection
2. Verify Firebase Firestore rules allow writes for authenticated users
3. Check browser console for permission errors
4. Try logging out and back in

#### Slides Not Uploading
1. **File too large:** Images over 1MB may fail; try compressing first
2. **Browser storage:** Clear cache and try again
3. **Firestore limits:** Firestore documents have 1MB limit; use external URLs for large images
4. **Network error:** Check internet connection stability

### Display Issues

#### Display Not Updating
1. **Check internet:** Display needs connection for Firebase sync
2. **Hard refresh:** Press `Ctrl + F5` (Windows) or `Cmd + Shift + R` (Mac)
3. **Clear cache:** Clear browser cache and reload
4. **Check Firebase:** Verify data exists in Firestore console

#### Display Freezes / Shows Old Data After TV Wake-Up
This is caused by the TV entering sleep/standby and killing the browser's WebSocket connection to Firebase.

1. **Automatic fix:** The built-in auto-sync keep-alive re-fetches data every 10 minutes — usually resolves itself within 10 min of the TV waking up
2. **Manual fix:** Hard refresh (`Ctrl + F5`) immediately re-establishes the connection
3. **Prevent sleep:** On the display device, set the OS power settings to **never sleep / never turn off display**
4. **Chrome flags:** Navigate to `chrome://settings/content/automaticDownloads` and ensure the display page is not being throttled

#### Prayer Times Not Showing
1. **API Settings:** Ensure city/country are set correctly in admin panel
2. **Network Error:** Check if `api.aladhan.com` is accessible
3. **CORS Issues:** Some networks block API requests; try different network
4. **Browser Console:** Check for API error messages (F12)

#### Jummah Not Showing on Friday
1. **Date Check:** Verify system date/time is correct
2. **Settings Saved:** Ensure Jummah times were saved in admin panel
3. **Browser Refresh:** Reload the page
4. **Console Errors:** Check browser console (F12) for JavaScript errors

#### Slideshow Not Working
1. **No Slides:** Add at least one slide in admin panel
2. **All Hidden:** Check that slides have visibility enabled (eye icon)
3. **Loading:** Look for "Loading slides..." message
4. **Image URLs:** Verify image URLs are accessible
5. **Console Errors:** Check for image load errors in console (F12)

#### Wrong Theme/Layout
1. **Admin Save:** Ensure you clicked "Save Changes" after selecting theme/layout
2. **Cache:** Hard refresh the display page (`Ctrl + F5`)
3. **Database:** Check Firestore console for `theme` and `layout` values

### Performance Issues

#### Slideshow Stuttering
1. **Reduce image sizes:** Re-upload images at lower resolution
2. **Close other programs:** Free up system resources
3. **Hardware acceleration:** Enable in browser settings
4. **Update browser:** Use latest version of Chrome or Edge

#### Page Loading Slowly
1. **Network Speed:** Check internet connection speed
2. **Too Many Slides:** Remove unused slides from collection
3. **Firebase Region:** Ensure Firestore region is geographically close

---

## Security Best Practices

### Firebase Configuration Security

#### Current Setup (Development)
Firebase credentials are currently hardcoded in both HTML files. This is acceptable for:
- Local network displays
- Low-security public displays
- Development/testing environments

#### Production Security Options

**Option 1: Firebase Hosting (Recommended)**

Firebase Hosting provides automatic credential protection:

1. Install Firebase tools:
   ```bash
   npm install -g firebase-tools
   ```

2. Initialize Firebase Hosting:
   ```bash
   firebase login
   firebase init hosting
   ```

3. Deploy files:
   ```bash
   firebase deploy --only hosting
   ```

4. Access via Firebase URL:
   - `https://your-project-id.web.app/`

**Advantages:**
- Credentials protected by reserved URLs
- Free SSL certificate
- CDN for faster loading
- Automatic security headers

**Option 2: Environment Variables (Advanced)**

For custom hosting with build tools:

1. Install dotenv package
2. Create `.env` file (add to `.gitignore`)
3. Reference variables in build process
4. Use build tool (webpack, rollup) to inject at build time

**Option 3: Backend Proxy**

Route Firebase requests through your own server:

1. Create backend API endpoint
2. Store credentials server-side
3. Frontend requests go to your API
4. Your API communicates with Firebase

### Admin Access Security

1. **Strong Passwords:**
   - Use 16+ character passwords
   - Include uppercase, lowercase, numbers, symbols
   - Consider using a password manager

2. **Email Security:**
   - Use dedicated admin email
   - Enable 2-factor authentication on email account
   - Don't share credentials

3. **Network Security:**
   - Access admin panel from secure networks only
   - Avoid public WiFi
   - Use VPN if accessing remotely

4. **Regular Updates:**
   - Change admin password every 90 days
   - Review Firebase authentication logs
   - Remove unused admin accounts

### Firestore Security Rules

Keep these rules to ensure display is public but settings are protected:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Public read, authenticated write
    match /{document=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

---

## Advanced Features

### Kiosk Mode Setup

#### Windows

**Option 1: Chrome Kiosk Mode**
1. Create shortcut with target:
   ```
   "C:\Program Files\Google\Chrome\Application\chrome.exe" --kiosk --app=file:///C:/path/to/index.html
   ```
2. Add to Windows Startup folder

**Option 2: Auto-Launch on Boot**
1. Press `Win + R`, type `shell:startup`
2. Create shortcut to Chrome kiosk command
3. Restart computer to test

#### Raspberry Pi

**Full Kiosk Setup:**

1. Install Chromium:
   ```bash
   sudo apt-get update
   sudo apt-get install chromium-browser unclutter
   ```

2. Edit autostart:
   ```bash
   sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
   ```

3. Add these lines:
   ```bash
   @xset s off
   @xset -dpms
   @xset s noblank
   @chromium-browser --noerrdialogs --disable-infobars --kiosk file:///home/pi/index.html
   @unclutter -idle 0
   ```

4. Reboot:
   ```bash
   sudo reboot
   ```

### Custom Branding

#### Change Mosque Name
Edit in `index.html` around line 1187:
```html
<h1>Your Mosque Name</h1>
<span>Your Location</span>
```

#### Change Colors (Beyond Themes)

Edit CSS variables in `index.html` around line 20:
```css
:root {
    --gold-primary: #d4af37;  /* Main accent color */
    --gold-light: #f3e5ab;    /* Light variant */
    --gold-dark: #aa8c2c;     /* Dark variant */
    --text-white: #ffffff;     /* Main text */
    --text-muted: #8b9bb4;    /* Secondary text */
}
```

### Multiple Displays

Run different configurations on multiple screens:

1. Use same Firebase project for all displays
2. Each screen loads `index.html`
3. Change theme/layout per screen by:
   - Creating separate HTML files with different default configs
   - OR using URL parameters (requires custom JavaScript)

### Backup and Recovery

#### Export Settings
1. Go to Firebase Console
2. Firestore Data → Export/Import
3. Select collections to back up
4. Download JSON

#### Import Settings
1. Upload JSON file in Firestore console
2. OR manually re-enter in admin panel

---

## Support and Maintenance

### Regular Maintenance Tasks

**Weekly:**
- Check display is running correctly
- Verify prayer times are accurate
- Update slides as needed

**Monthly:**
- Update admin password
- Review Firebase usage/quota
- Check for browser updates

**Quarterly:**
- Backup Firestore data
- Review and update API settings
- Clear unused slides

### System Requirements Check

Periodically verify:
- [ ] Internet connection is stable
- [ ] Display screen is functioning
- [ ] Browser is up to date
- [ ] Firebase project is active
- [ ] Aladhan API is accessible

---

## Credits

**Developed for:** Faizan-E-Madinah & Education Center, Ravensthorpe  
**Prayer Times API:** [Aladhan API](https://aladhan.com)  
**Cloud Platform:** Google Firebase  
**Fonts:** Google Fonts (Montserrat, Amiri, Noto Nastaliq Urdu)  
**Icons:** Font Awesome 6.4.0  

---

## Version History

**Current Version:** 3.0  
**Last Updated:** February 2026

### Changelog

**v3.0 (Feb 2026)**
- ✅ Added bilingual English + Urdu prayer names, headers, section titles, and countdown overlays (Noto Nastaliq Urdu font)
- ✅ Added 4-week Jamaat schedule (weekly tabs in admin, auto-selected on display by date)
- ✅ Added configurable countdown-to-Jamaat duration (admin input, 1–60 min, default 15)
- ✅ Added 5th **Custom Color** theme with live RGB hex picker in admin
- ✅ Added auto-sync keep-alive (force re-fetch from Firestore server every 10 min to survive TV sleep)
- ✅ Added monthly prayer timetable preview in admin API Settings tab
- ✅ Updated Firestore schema (`weeklyPrayers`, `countdownMinutes`, `customColor`)

**v2.0 (Feb 2026)**
- ✅ Fixed duplicate school value assignment
- ✅ Removed leftover Jummah DOM references
- ✅ Added Firebase security documentation
- ✅ Improved error handling

**v1.0 (Jan 2026)**
- Initial release with full feature set

---

## Quick Reference

### Common Admin Tasks

| Task | Steps |
|------|-------|
| Change prayer times (all weeks) | Admin → Timings → W1/W2/W3/W4 tabs → Edit times → Save |
| Change countdown duration | Admin → Timings → Countdown Minutes field → Save |
| Add slide | Admin → Slides → Upload/URL → Add |
| Change theme | Admin → Appearance → Select theme → Save |
| Set custom brand colour | Admin → Appearance → Custom Color card → Pick colour → Save |
| Update location | Admin → API Settings → Edit city/country → Save |
| Preview monthly timetable | Admin → API Settings → Load Timetable button |

### Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Fullscreen (Windows/Linux) | `F11` |
| Fullscreen (Mac) | `Cmd + Ctrl + F` |
| Exit Fullscreen | `Esc` |
| Browser Console | `F12` |
| Hard Refresh | `Ctrl + F5` (Win) / `Cmd + Shift + R` (Mac) |

### Firebase Console Links

- **Firestore Database:** `https://console.firebase.google.com/project/YOUR_PROJECT_ID/firestore`
- **Authentication:** `https://console.firebase.google.com/project/YOUR_PROJECT_ID/authentication`
- **Project Settings:** `https://console.firebase.google.com/project/YOUR_PROJECT_ID/settings/general`

---

## Need Help?

If you encounter issues not covered in this guide:

1. **Check browser console** (F12) for error messages
2. **Review Firebase Firestore** rules and data
3. **Test with different browser** (Chrome recommended)
4. **Verify internet connection** stability
5. **Check Aladhan API status** at https://aladhan.com

---

*This system is designed to be maintenance-free once configured. Enjoy your automated mosque display!*
