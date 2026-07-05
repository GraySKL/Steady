# Steady — EDS & Dysautonomia Tracking App

## ⚠️ IMPORTANT LEGAL DISCLAIMER

**Steady is NOT a medical device and does NOT provide medical advice.** It is a personal health tracking tool designed to help you monitor your own symptoms and patterns. 

- **Not a diagnosis tool.** Stand test results are for personal reference only; they do not diagnose POTS or any other condition.
- **Not a replacement for medical care.** Always consult a healthcare provider for symptoms, medication changes, or diagnosis.
- **Data accuracy.** While we work to ensure accuracy, this app makes no guarantees about data completeness or correctness.
- **Your responsibility.** You are responsible for verifying any data imported from external sources (Fitbit, Google Health, etc.) and understanding how it's used.
- **No warranties.** This app is provided "as-is" with no warranty of any kind, express or implied.

See the [Full Legal Disclaimer](#full-legal-disclaimer) section below for complete terms.

---

## Table of Contents
1. [About This App](#about-this-app)
2. [Why It Exists](#why-it-exists)
3. [How to Use Steady](#how-to-use-steady)
4. [Installation](#installation)
5. [Features](#features)
6. [Developers Guide](#developers-guide)
7. [Architecture](#architecture)
8. [Building from Source](#building-from-source)
9. [Testing](#testing)
10. [Contributing](#contributing)
11. [Full Legal Disclaimer](#full-legal-disclaimer)
12. [Privacy & Data Handling](#privacy--data-handling)
13. [License & Attribution](#license--attribution)

---

## About This App

**Steady** is a mobile health tracking application designed for people with Ehlers-Danlos Syndrome (EDS) and dysautonomia, particularly Postural Orthostatic Tachycardia Syndrome (POTS). It runs entirely offline on your Android phone, never sends data anywhere, and lets you track what matters: heart rate changes when standing, symptom patterns, triggers, and how you feel day-to-day.

**Platform:** Android 10+  
**License:** [Choose: MIT, Apache 2.0, GPL, etc.]  
**Current Version:** 1.1  
**Built With:** Capacitor 6, Vanilla JavaScript, Android Studio/Gradle

---

## Why It Exists

### The Problem
People with POTS and EDS need to spot flare patterns and understand what makes their condition worse. Existing health apps either:
- Require cloud sync (privacy concern for sensitive health data)
- Focus on fitness, not symptom tracking
- Don't understand the unique needs of dysautonomia (e.g., the POTS stand test)
- Add cognitive overhead when you're already fatigued

### The Solution
Steady solves this by:
- **Living entirely on your phone** — zero network calls, zero cloud dependency
- **Tracking what you care about:** POTS stand tests, daily symptoms, triggers, mood
- **Pulling in existing health data** from Fitbit/Google Health exports (optional)
- **Showing patterns** through trends charts (resting HR over time, sleep quality)
- **Keeping it simple** — tap a button to log how you're feeling, not a 50-field form
- **Works offline** — airplane mode, no signal, no problem

### For Developers
This is also a reference implementation of:
- A single-file web app bundled into a native Android shell (Capacitor)
- Privacy-first mobile health tracking
- Custom native plugin integration for file I/O
- Symptom tracking + data visualization

---

## How to Use Steady

### Getting Started

1. **Install** the APK from [releases](#installation)
2. **Open the app** — you'll see the "Today" screen with a heart icon
3. **Start tracking** by tapping buttons

### Core Features

#### The POTS Stand Test
A guided 10-minute test for POTS diagnosis:
1. Tap **"Run stand test"** on the Today screen
2. **Rest for 5 minutes**, tapping to confirm when you're lying or sitting
3. **Record your resting heart rate**
4. **Stand up** and record your HR at 1, 3, 5, and 10 minutes
5. The app **flags elevated readings** (≥+30 bpm for adults, ≥+40 for teens)

#### Log Symptoms & Triggers
Tap **"Log something"** to record:
- **What you're feeling:** brain fog, dizziness, fatigue, pain, etc.
- **What might have caused it:** stairs, heat, caffeine, poor sleep, etc.
- **How severe:** scale of 1–5

#### Daily Check-In
Tap the **weather-face button** (Today screen) to record how you're feeling overall (1–5). This feeds the insight engine — it needs ≥3 good days and ≥3 rough days to find patterns.

#### View Trends
Tap **"Trends"** to see:
- **Resting heart rate over time** (line chart)
- **Sleep quality** (bar chart)
- **Step count** (daily steps, if imported)
- **Insight cards** ("Your resting HR ran 8 bpm higher on rough days")

#### Import Fitbit / Google Health Data
1. Go to **Backup** (top-right menu) → **"Import Fitbit data"**
2. Either:
   - Tap **"Find my download for me"** (scans your Downloads folder)
   - Tap **"Import Fitbit data"** (pick the file yourself)
3. Choose your `takeout-*.zip` from a Google Takeout export
4. The app pulls in: resting HR, sleep, step count, HRV (if available)
5. **Your data stays on your phone** — nothing is uploaded

#### Back Up Your Data
1. Go to **Backup** → **"Back up (download a file)"**
2. A JSON file downloads to your phone's Downloads folder
3. **Keep it safe** — it's your complete data backup
4. To restore: **Backup** → **"Restore from a backup"** and pick the file

#### Settings
Tap **Backup** and scroll to toggle:
- **"I'm under 19"** — changes stand-test threshold to +40 bpm (pediatric POTS diagnostic threshold)

---

## Installation

### From Pre-Built APK (Easiest)

1. **Download** the latest `Steady-vX.X.apk` from [GitHub Releases](https://github.com/your-org/steady/releases)
2. **Transfer to your phone** (email, Google Drive, USB cable)
3. **Open the file** → **Install** → tap "Install unknown apps" if prompted
4. **Done** — the app is now on your phone

### From Source (Developers)

See [Building from Source](#building-from-source) below.

---

## Features

| Feature | Description |
|---------|-------------|
| **POTS Stand Test** | Guided 10-minute test with HR tracking and threshold alerts (+30/+40 bpm) |
| **Symptom Logging** | Quick tap-to-log interface for symptoms and triggers |
| **Daily Mood Check-In** | 1–5 scale to track how you're feeling |
| **Trends Visualization** | Charts for resting HR, sleep, steps over time |
| **Pattern Insights** | Auto-compare good vs. rough days to spot correlations |
| **Fitbit/Google Health Import** | Read-only import from Takeout exports (no cloud sync) |
| **Backup & Restore** | JSON export/import for data portability |
| **Offline-First** | Zero network calls; all data stays on your phone |
| **Age-Aware Thresholds** | Switch between adult (+30 bpm) and teen (+40 bpm) POTS criteria |

---

## Developers Guide

### Project Structure

```
steady/                          # Web app source
├── steady.html                  # Single-file web app (all HTML/CSS/JS)
├── HANDOFF.md                   # Build setup & troubleshooting
├── DOCUMENTATION.md             # This file
└── tests/                       # Unit tests for ZIP parsing, data model

steady-android/                  # Capacitor Android project
├── www/index.html               # Copy of steady.html bundled in the APK
├── android/                     # Android Studio project
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── AndroidManifest.xml
│   │   │   ├── java/com/sara/steady/
│   │   │   │   ├── MainActivity.java
│   │   │   │   └── FilePickerPlugin.java
│   │   │   └── res/              # Icons, strings, colors
│   │   └── build.gradle
│   ├── build.gradle
│   └── local.properties          # SDK path
├── capacitor.config.json
├── package.json
└── test-build.ps1               # Build verification script
```

### Key Components

#### 1. **steady.html** — The Whole App
A 1,300-line vanilla JavaScript web app. No frameworks, no bundler, no dependencies.

**Key sections:**
- **State model** (line ~50): `state = { standTests, logs, feelings, settings, health, ... }`
- **Storage** (line ~100): localStorage + JSON backup/restore
- **POTS stand test** (line ~620): timer, HR input, threshold logic
- **Symptom logging** (line ~750): chips UI for quick tapping
- **Trends** (line ~900): SVG charts (no charting library)
- **Native bridge** (line ~1100): `window.Capacitor` calls to `FilePickerPlugin`
- **UI rendering** (line ~500+): DOM manipulation, no virtual DOM

**Why vanilla JS?**
- Single file = easy to bundle & audit
- No build step = faster development
- Smaller APK (no runtime overhead)
- CSP-safe (no eval, no dynamic script loading)

#### 2. **FilePickerPlugin.java** — Native Integration
Bridges JavaScript to Android's file system.

**What it does:**
- `pickFile()` → opens system file chooser (ACTION_GET_CONTENT)
- `readFileChunk(uri, offset, length)` → reads 4 MB at a time (big files don't OOM the bridge)
- `readFileUri(uri)` → reads a full file as base64
- `saveToDownloads(name, mime, data)` → writes to MediaStore.Downloads
- `checkStoragePermission()` → queries MANAGE_EXTERNAL_STORAGE
- `openStorageSettings()` → launches the permission settings page

**Why chunked reads?**
A 119 MB Fitbit export as one base64 string would crash the Capacitor bridge. We read in 4 MB chunks and reassemble on the JS side.

#### 3. **MainActivity.java** — App Shell
Capacitor's BridgeActivity subclass. Registers the plugin at the right time (`onCreate` before `super.onCreate()`).

#### 4. **Test Suite** (tests/)
Unit tests for:
- ZIP parser (Deflate decompression)
- Data import (reading Fitbit Takeout structure)
- Calculation logic (POTS thresholds)

All synthetic test data; no real PHI.

---

## Architecture

### Data Model

```javascript
state = {
  standTests: [
    {
      date: "2026-01-15",
      resting: 68,
      readings: [ 92, 95, 98 ],  // at 1, 3, 5 min
      reading10m: 96,
      phase: "done",
      notes: ""
    },
    ...
  ],
  logs: [
    {
      date: "2026-01-15",
      time: "14:32",
      symptoms: [ "brain fog", "dizziness" ],
      triggers: [ "stairs", "heat" ],
      severity: 4,
      notes: ""
    },
    ...
  ],
  feelings: [
    { date: "2026-01-15", rating: 3 },  // 1–5
    ...
  ],
  health: {
    restingHR: { "2026-01-15": 72, ... },   // from Fitbit import
    sleepMin: { "2026-01-15": 420, ... },   // minutes
    steps: { "2026-01-15": 5432, ... },
    hrv: { ... },                           // if available
    lastImport: "2026-01-15T14:32:00Z"
  },
  settings: {
    ageGroup: "adult"  // or "teen"
  }
}
```

All data lives in `localStorage` (no server). Data is keyed by ISO date strings (`YYYY-MM-DD`) so it's timezone-safe.

### Storage Pipeline

1. **In-memory state** (JavaScript object)
2. **localStorage** (auto-saved after every action)
3. **Backup export** (JSON file to Downloads)
4. **Fitbit import** (ZIP → ZIP parser → state.health)

### Permissions (Android 10+)

| Permission | Why | Required? |
|-----------|-----|-----------|
| `READ_EXTERNAL_STORAGE` | Pick files | ✅ Yes (API ≤32) |
| `MANAGE_EXTERNAL_STORAGE` | Scan Downloads for auto-import | Optional (nice-to-have) |
| `INTERNET` | Removed in v1.1 | ❌ No (app is offline) |

### Capacitor Plugins Used

| Plugin | Version | Purpose |
|--------|---------|---------|
| `@capacitor/core` | 6 | Bridge to native code |
| `@capacitor/android` | 6 | Android runtime |
| `@capacitor/filesystem` | 6 | Read Download folder (optional) |
| `FilePickerPlugin` | custom | File picker + chunked reads |

---

## Building from Source

### Prerequisites

- **Windows, macOS, or Linux**
- **Node.js** v18+
- **JDK 17** (for Android builds)
- **Android SDK** (API level 34)
- **Gradle** (bundled with the Android project)

### Step 1: Set Up the Toolchain

#### macOS / Linux
```bash
# Install JDK 17 (if not already present)
brew install openjdk@17

# Download Android SDK command-line tools from:
# https://developer.android.com/studio/command-line-tools
# Extract to ~/android-sdk (or your preferred location)

# Accept licenses
mkdir -p ~/android-sdk/licenses
echo -e "\n8933bad161af4178b1185d1a37fbf41ea5269c55\n" > ~/android-sdk/licenses/android-sdk-license
echo -e "\nd56f5187479451eabf01fb78af6dfcb131a6481e\n" >> ~/android-sdk/licenses/android-sdk-license

# Install build tools
~/android-sdk/cmdline-tools/latest/bin/sdkmanager \
  "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

#### Windows (PowerShell)
```powershell
# Download JDK 17 from https://adoptium.net
# Install to C:\jdk-17 (or your preferred location)

# Download Android SDK from https://developer.android.com/studio/command-line-tools
# Extract to C:\android-sdk

# Accept licenses
$lic="C:\android-sdk\licenses"
New-Item -ItemType Directory -Force $lic | Out-Null
[System.IO.File]::WriteAllText("$lic\android-sdk-license",
  "`n8933bad161af4178b1185d1a37fbf41ea5269c55`nd56f5187479451eabf01fb78af6dfcb131a6481e")

# Install build tools
$sdk="C:\android-sdk"
& "$sdk\cmdline-tools\latest\bin\sdkmanager.bat" --sdk_root=$sdk `
  "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

### Step 2: Clone & Prepare the Project

```bash
git clone https://github.com/your-org/steady.git
cd steady/steady-android

# Install Node dependencies
npm install

# Create local.properties with SDK path
# macOS / Linux:
echo "sdk.dir=$HOME/android-sdk" > android/local.properties

# Windows:
# Create android\local.properties manually with:
#   sdk.dir=C:/android-sdk
```

### Step 3: Build the APK

```bash
# macOS / Linux
export JAVA_HOME=/usr/libexec/java_home -v 17
export ANDROID_SDK_ROOT=$HOME/android-sdk
export ANDROID_HOME=$HOME/android-sdk

npx --yes cap sync android
cd android
./gradlew assembleDebug --no-daemon

# Windows (PowerShell)
$env:JAVA_HOME = "C:\jdk-17"
$env:ANDROID_SDK_ROOT = "C:\android-sdk"
$env:ANDROID_HOME = "C:\android-sdk"

npx --yes cap sync android
cd android
& ".\gradlew.bat" assembleDebug --no-daemon
```

The APK appears at:
```
steady-android/android/app/build/outputs/apk/debug/app-debug.apk
```

### Step 4: Install on Phone

**Option A: Via ADB**
```bash
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
```

**Option B: Manual**
1. Copy `app-debug.apk` to your phone (email, Google Drive, USB)
2. Open the file on your phone
3. Tap **Install** → confirm unknown app warning
4. Done

---

## Testing

### Unit Tests (Data & Logic)

The `tests/` folder contains Jest tests for:
- ZIP parsing (Deflate decompression)
- Fitbit data import logic
- POTS threshold calculations
- Date/timezone handling

```bash
npm test
```

### Manual Testing Checklist

#### Stand Test Flow
- [ ] Start a stand test
- [ ] Record resting HR
- [ ] Record HRs at 1, 3, 5, 10 minutes
- [ ] Verify threshold alert appears if HR exceeds limit
- [ ] Check data saves to localStorage
- [ ] Verify test shows in Today history

#### Logging
- [ ] Tap "Log something"
- [ ] Select symptoms and triggers
- [ ] Set severity
- [ ] Log saves to Today view
- [ ] Log appears in Logs tab

#### Trends
- [ ] With ≥10 stand tests, resting HR chart appears
- [ ] Tap a date in the chart
- [ ] Verify mood rating + severity breakdown for that day

#### Backup / Restore
- [ ] Tap **Backup** → **"Back up"** → file downloads
- [ ] Uninstall app
- [ ] Reinstall
- [ ] Tap **Restore** → pick the backup file
- [ ] All data reappears

#### Fitbit Import
- [ ] Grab a `takeout-*.zip` from Google Takeout (Fitbit section)
- [ ] Tap **Backup** → **"Import Fitbit data"**
- [ ] Pick the file
- [ ] Verify resting HR and sleep data loads
- [ ] Check Trends charts update

### End-to-End Test (Automated)

The build script (`test-build.ps1` on Windows) runs:
1. **JS syntax check** — `node --check` on the bundled script
2. **APK build** — `gradle assembleDebug`
3. **Duplicate method check** — catches Java compilation errors

---

## Contributing

### Found a Bug?
Open an issue with:
- **Describe it:** what did you expect vs. what happened?
- **Reproduce it:** step-by-step
- **Environment:** Android version, phone model, app version
- **Logs:** if available (look for `adb logcat` output)

### Want to Add a Feature?
1. **Open an issue first** — let's discuss
2. **Fork the repo**
3. **Create a branch** (`feature/my-feature`)
4. **Edit `steady.html`** (the source of truth)
5. **Run tests** (`npm test`)
6. **Test on a real device** (or emulator)
7. **Submit a PR** with a clear description

### Code Style

**JavaScript (steady.html):**
- No frameworks, no transpilation
- Vanilla ES6 (for/const/let/arrow functions OK)
- Comments only for **why**, not **what**
- Keep functions small and focused

**Java (FilePickerPlugin.java):**
- Follow Android conventions
- Use AndroidX libraries (no support library)
- Add `@PluginMethod` to public entry points
- Catch exceptions; never let them crash the app

### Before Submitting a PR

- [ ] **Syntax check:** `node --check` on your JS additions
- [ ] **Test on device:** Install APK and manually test your feature
- [ ] **No network calls:** app must stay offline
- [ ] **No sensitive data in logs:** use `Log.d()` sparingly
- [ ] **Commit message:** one line, clear intent ("Add symptom severity slider" not "fixes bug")

---

## Roadmap

**v1.1 (Current)** — File import fix, chunked reads, native backup

**v1.2 (Proposed)**
- [ ] Period tracking (for POTS/EDS correlation with menstrual cycle)
- [ ] Medication log
- [ ] Export to CSV (for doctor visits)
- [ ] Dark mode (respects system preference)

**v2.0 (Future)**
- [ ] Sync across multiple devices (end-to-end encrypted, optional)
- [ ] Wearable integration (direct HR import, not just Fitbit export)
- [ ] Machine learning: auto-detect flare patterns

---

---

## Full Legal Disclaimer

### 1. NOT MEDICAL ADVICE OR A MEDICAL DEVICE

**Steady is not a medical device and does not provide medical advice, diagnosis, or treatment.** It is a personal health tracking application designed for informational purposes only. The information provided by Steady:

- Is **not a substitute** for professional medical advice, diagnosis, or treatment
- Should **never be used** to diagnose, prevent, treat, or manage any medical condition
- Is **for personal reference only** and not intended to replace clinical judgment or medical consultation
- May contain **inaccuracies** and should not be relied upon as the sole basis for any health decision

**You should always consult with a qualified healthcare provider** before making any health-related decisions, starting or stopping medications, or making changes to your treatment plan.

### 2. STAND TEST RESULTS

The "POTS Stand Test" feature in Steady is **not a diagnostic tool**. Results do not diagnose Postural Orthostatic Tachycardia Syndrome (POTS) or any other condition. The threshold alerts (+30 bpm for adults, +40 for teens) are based on general medical literature but:

- Do not constitute a diagnosis
- Are **not a substitute** for evaluation by a cardiologist or autonomic specialist
- May not apply to your individual situation
- Should be discussed with your healthcare provider if concerning

**POTS diagnosis requires clinical evaluation by a qualified physician.**

### 3. DATA ACCURACY & COMPLETENESS

While we strive for accuracy, Steady makes **no warranties** about:
- The accuracy or completeness of imported health data (from Fitbit, Google Health, etc.)
- The correctness of calculations or interpretations
- The availability or reliability of the app
- The safety or integrity of your data

**You are responsible for:**
- Verifying imported data matches the original source
- Understanding how your health data is used
- Keeping backups of important health information
- Reporting inaccuracies or concerns to your healthcare provider

### 4. DATA PRIVACY & SECURITY

Steady is designed with privacy in mind:
- **All data stays on your phone** — no information is transmitted to servers
- **No cloud sync** — your data is not uploaded, shared, or backed up remotely
- **No tracking** — the app does not collect usage statistics or analytics
- **No advertisements** — the app contains no ads, cookies, or third-party trackers

**However:**
- You are responsible for **securing your phone** against unauthorized access
- If your phone is lost or stolen, **you may lose your health data** unless you have backed it up
- **Backups are plain-text JSON files** — protect backup files as you would any sensitive health information
- We are **not responsible** for data loss due to device failure, theft, or accidents

### 5. NO WARRANTY

**Steady is provided "AS IS" without warranty of any kind**, express or implied, including but not limited to:
- Merchantability, fitness for a particular purpose, or non-infringement
- That the app will be error-free, uninterrupted, or secure
- That the app will meet your needs or expectations
- That bugs or defects will be fixed

### 6. LIMITATION OF LIABILITY

**Under no circumstances shall the authors or maintainers of Steady be liable** for any direct, indirect, incidental, special, or consequential damages arising from:
- Use or inability to use the app
- Data loss or corruption
- Reliance on information provided by the app
- Any health-related consequences
- Any other cause related to the app

This applies even if advised of the possibility of such damages.

### 7. HEALTH & SAFETY

- **Do not rely on Steady alone** for health management. Use it as a tool alongside medical care from qualified healthcare providers.
- **Report concerning symptoms** to a healthcare provider immediately, not just to the app.
- **In emergencies, call emergency services** (911 in the US) — do not wait for the app.
- **Be careful during stand tests.** If you feel faint, dizzy, or unwell, sit or lie down immediately.

### 8. THIRD-PARTY DATA

If you import data from Fitbit, Google Health, or other sources:
- **We do not control** how that data is collected, stored, or handled by third parties
- **You should review** the privacy policies of those services
- **You are responsible** for understanding the terms of service of any platform you use to export your data
- **We are not liable** for issues with third-party data or services

### 9. UPDATES & CHANGES

Steady may be updated or discontinued at any time. Updates may:
- Introduce new features or change existing ones
- Require rebuilding the app from source
- Be incompatible with older versions
- We do not guarantee **backward compatibility** with previous versions

### 10. GOVERNING LAW

This disclaimer is governed by the laws of [Jurisdiction], without regard to conflicts of law principles. By using Steady, you agree to resolve any disputes through binding arbitration or small-claims court.

---

## Privacy & Data Handling

### Your Data Stays on Your Phone

**Steady collects no data from you and sends no data anywhere.** Specifically:

- **No cloud sync.** Your data is never sent to servers or stored in the cloud
- **No analytics.** We do not track how you use the app
- **No advertising.** The app contains no ads, trackers, or third-party scripts
- **No user accounts.** You don't create an account; there are no usernames or passwords
- **No identification.** We don't know who you are or what you're tracking

### Data Storage

- **Device storage.** Your data is stored in your phone's `localStorage` (encrypted by Android)
- **Backups.** You can export backups as plain-text JSON files to your Downloads folder — **treat these like sensitive health records**
- **Imported data.** When you import a Fitbit export, **only the structured data is kept** (resting HR, sleep, steps); the original ZIP file is deleted after import

### Permissions

**Read Storage** (Android 12 and below): Needed to pick files from your phone when importing Fitbit data.  
**Manage All Files**: Optional; used only if you want the app to auto-scan your Downloads folder for Fitbit exports.  
**Internet**: **Removed in v1.1** — the OS itself prevents network access.

### What We Don't Do

- ❌ We don't sell, share, or disclose your data
- ❌ We don't use your data for marketing or research
- ❌ We don't track your location, contacts, or other apps
- ❌ We don't send crash reports unless you explicitly choose to via Android's system settings
- ❌ We don't have accounts, logins, or user databases

---

## License & Attribution

### Software License

Steady is released under the **[MIT License](LICENSE)**. You are free to:
- Use Steady for any purpose, commercial or personal
- Modify and distribute modified versions
- Include Steady in your own projects

**Under the condition that** you include the original copyright notice and license with any distribution.

### Third-Party Licenses

Steady uses the following open-source libraries (all included in the APK or build):

| Library | License | Purpose |
|---------|---------|---------|
| Capacitor | MIT | Mobile app framework |
| Capacitor Android | MIT | Android runtime for Capacitor |
| Capacitor Filesystem | MIT | File system access |
| Android Gradle | Apache 2.0 | Build toolchain |
| AndroidX | Apache 2.0 | Android compatibility libraries |

A complete list of dependencies and their licenses appears in `package.json` (for Node modules) and `android/build.gradle` (for Android libraries).

### Attribution

Built with care for people managing chronic illness. Special thanks to the EDS and POTS communities for the feedback that shaped this app.

If you use, fork, or build on Steady, we appreciate a mention in your project's documentation.

### Trademarks

- **Fitbit** is a trademark of Google LLC
- **Android** is a trademark of Google LLC
- **Google Health** / **Google Fit** are trademarks of Google LLC

This project is not affiliated with, endorsed by, or sponsored by Google or any other company.

---

## Questions?

- **For users:** See the [How to Use](#how-to-use-steady) section
- **For developers:** Check [HANDOFF.md](HANDOFF.md) for build troubleshooting
- **Medical questions:** Consult a healthcare provider
- **Found a bug or security issue?** Open a private issue on GitHub (don't post security vulnerabilities publicly)

---

## Acknowledgments

Built with care for people managing chronic illness. Steady is a personal project designed to help you understand your own health better.

**A note to users:** You are the expert on your own body. Use Steady as a tool to spot your patterns and communicate with your healthcare team — but trust your own instincts and always seek professional medical advice for health concerns.

---

**Last updated:** 2026-07-04  
**Version:** 1.1  
**Maintained by:** [Your Name / Organization]  
**GitHub:** https://github.com/your-org/steady  
**License:** MIT (see [LICENSE](LICENSE) file)


---

# Appendix: Developer sections relocated from the README

*(Moved here 2026-07-05 when the README was rewritten for non-technical users. Text preserved as-is.)*

## What's in This Repo

```
steady/
â”œâ”€â”€ steady.html                  # The entire app (single-file vanilla JS)
â”œâ”€â”€ DOCUMENTATION.md             # Complete user & developer guide
â”œâ”€â”€ HANDOFF.md                   # Build setup, toolchain, troubleshooting
â”œâ”€â”€ README.md                    # User-facing intro
â”œâ”€â”€ LICENSE                      # MIT License
â”œâ”€â”€ manifest.webmanifest         # PWA manifest (web install)
â”œâ”€â”€ sw.js                        # Service worker (offline caching)
â”œâ”€â”€ icons/                       # PWA icons
â””â”€â”€ .github/workflows/pages.yml  # GitHub Pages deployment
```

The Capacitor Android project (`steady-android/`) is maintained separately; see [BUILDING.md](BUILDING.md) for how to recreate it.

## Architecture

### Single-File Web App

`steady.html` is **the entire user-facing app** â€” vanilla JavaScript with:
- No frameworks, no build step, no dependencies
- Direct DOM manipulation (no virtual DOM)
- SVG charts (no charting library)
- localStorage for persistence
- Native bridge calls to `FilePickerPlugin` for file access

**Why?**
- âœ… Easy to audit for security
- âœ… Small APK size
- âœ… Fast load time
- âœ… Works offline by default

### Native Plugin (FilePickerPlugin.java)

Bridges the JavaScript app to Android's file system:
- `pickFile()` â€” Opens system file chooser
- `readFileChunk()` â€” Reads large files in 4 MB chunks (no bridge OOM)
- `saveToDownloads()` â€” Saves backups to MediaStore.Downloads

### Capacitor

[Capacitor 6](https://capacitorjs.com) wraps the web app in a native Android shell:
- Bundles the HTML/CSS/JS inside the APK
- Provides `@capacitor/filesystem` for directory access
- Bridges JavaScript â†” Native code

## Data Model

All data lives in **localStorage** (no server). Structure:

```javascript
{
  standTests: [ /* POTS stand test results */ ],
  logs: [ /* Symptom & trigger logs */ ],
  feelings: [ /* Daily 1-5 mood ratings */ ],
  medications: { /* daily check-off timestamps, keyed by date */ },
  health: { /* Imported wearable data */
    restingHR: { "2026-01-15": 72, ... },
    sleepMin: { "2026-01-15": 420, ... },
    sleepHR: { "2026-01-15": 63, ... },
    steps: { "2026-01-15": 5432, ... }
  },
  settings: { ageGroup: "adult", userName: "", reminders: [ /* user-defined */ ] }
}
```

**Privacy:** Your data never leaves your phone. **Backups** are plain-text JSON â€” treat them like sensitive medical records.

## FAQ (from the original README)

**Q: Why is there no cloud sync?**
A: Your health data is sensitive. Cloud sync requires an always-on server and introduces privacy/security risks. Backup â†’ Restore works fine for moving across devices and is under *your* control.

**Q: How do I delete my data?**
A: Uninstall the app (which clears localStorage). Backups are JSON files in your Downloads â€” delete them manually.

**Q: What if Fitbit changes their export format?**
A: The ZIP parser is in `steady.html` and can be updated. Open an issue if an export stops working.

**Q: Is my data safe if my phone is stolen?**
A: Steady data is protected by Android's storage encryption, but keep your phone's PIN/biometric lock enabled. Regular backups (stored safely) are your safety net.

