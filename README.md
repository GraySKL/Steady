# Steady — Personal Health Tracking for EDS & Dysautonomia

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform: Android](https://img.shields.io/badge/Platform-Android%2010%2B-green)](https://www.android.com)
[![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen)](HANDOFF.md)

A **completely offline**, private health tracking app for people with Ehlers-Danlos Syndrome (EDS) and dysautonomia, built with vanilla JavaScript and Android.

---

## ⚠️ LEGAL DISCLAIMER

**Steady is NOT a medical device and does NOT provide medical advice.** It is a personal health tracking tool for your own reference.

- **Not a diagnosis tool.** POTS stand test results are for personal monitoring only.
- **Not a replacement for medical care.** Always consult a healthcare provider.
- **No warranties.** This app is provided "as-is" with no guarantees.

**See [DOCUMENTATION.md](DOCUMENTATION.md#full-legal-disclaimer) for the complete legal disclaimer.**

---

## Features

✅ **POTS Stand Test** — Guided 10-minute test with heart rate tracking  
✅ **Symptom & Trigger Logging** — Quick tap-to-log interface  
✅ **Daily Mood Check-In** — Track how you're feeling  
✅ **Trends & Insights** — Charts + auto-comparison of good vs. rough days  
✅ **Fitbit/Google Health Import** — Read your existing health data (optional)  
✅ **100% Offline** — Zero network calls; your data never leaves your phone  
✅ **Backup & Restore** — Export/import your data as JSON  
✅ **Privacy First** — No accounts, no tracking, no ads  

---

## Quick Start

### For Users

1. **Download** the latest APK from [Releases](https://github.com/your-org/steady/releases)
2. **Install on your Android phone** (10+)
3. **Open the app** and start logging

See [**How to Use Steady**](DOCUMENTATION.md#how-to-use-steady) for detailed instructions.

### For Developers

1. **Clone this repo**
   ```bash
   git clone https://github.com/GraySKL/Steady.git
   cd Steady
   ```

2. **Choose your path:**

   **🔧 Build for Android** (want to contribute to the native app?)
   ```bash
   cd android-build
   ./test-build.ps1          # Windows
   # or: npx cap sync android && cd android && ./gradlew assembleDebug
   ```
   See [**BUILDING.md**](BUILDING.md) for detailed setup, troubleshooting, and development workflow.

   **💻 Edit the web app** (want to add features?)
   ```bash
   # Edit steady.html (the entire app is one file)
   # Then rebuild (see above) or test in a web browser
   ```
   See [**DOCUMENTATION.md**](DOCUMENTATION.md#developers-guide) for architecture details.

---

## What's in This Repo

```
steady/
├── steady.html                  # The entire app (single 1,300-line file)
├── DOCUMENTATION.md             # Complete user & developer guide
├── HANDOFF.md                   # Build setup, toolchain, troubleshooting
├── README.md                    # This file
├── LICENSE                      # MIT License
├── .gitignore
│
├── steady-android/              # Capacitor Android project
│   ├── www/index.html           # Copy of steady.html for bundling
│   ├── package.json
│   ├── capacitor.config.json
│   ├── test-build.ps1           # Build verification script (Windows)
│   │
│   └── android/                 # Android Studio project
│       ├── app/
│       │   ├── src/main/
│       │   │   ├── AndroidManifest.xml
│       │   │   ├── java/com/sara/steady/
│       │   │   │   ├── MainActivity.java
│       │   │   │   └── FilePickerPlugin.java
│       │   │   └── res/
│       │   └── build.gradle
│       ├── build.gradle
│       └── local.properties
│
└── tests/                       # Unit tests
    ├── zipParser.test.js
    ├── dataImport.test.js
    └── calculations.test.js
```

---

## Architecture

### Single-File Web App

`steady.html` is **the entire user-facing app** — 1,300 lines of vanilla JavaScript with:
- No frameworks, no build step, no dependencies
- Direct DOM manipulation (no virtual DOM)
- SVG charts (no charting library)
- localStorage for persistence
- Native bridge calls to `FilePickerPlugin` for file access

**Why?**
- ✅ Easy to audit for security
- ✅ Small APK size
- ✅ Fast load time
- ✅ Works offline by default

### Native Plugin (FilePickerPlugin.java)

Bridges the JavaScript app to Android's file system:
- `pickFile()` — Opens system file chooser
- `readFileChunk()` — Reads large files in 4 MB chunks (no bridge OOM)
- `saveToDownloads()` — Saves backups to MediaStore.Downloads
- `checkStoragePermission()` — Checks MANAGE_EXTERNAL_STORAGE
- `openStorageSettings()` — Opens permission settings

### Capacitor

[Capacitor 6](https://capacitorjs.com) wraps the web app in a native Android shell:
- Bundles the HTML/CSS/JS inside the APK
- Provides `@capacitor/filesystem` for directory access
- Bridges JavaScript ↔ Native code

---

## Data Model

All data lives in **localStorage** (no server). Structure:

```javascript
{
  standTests: [ /* POTS stand test results */ ],
  logs: [ /* Symptom & trigger logs */ ],
  feelings: [ /* Daily 1-5 mood ratings */ ],
  health: { /* Imported Fitbit data */
    restingHR: { "2026-01-15": 72, ... },
    sleepMin: { "2026-01-15": 420, ... },
    steps: { "2026-01-15": 5432, ... }
  },
  settings: { ageGroup: "adult" /* or "teen" */ }
}
```

**Privacy:** Your data never leaves your phone. **Backups** are plain-text JSON — treat them like sensitive medical records.

---

## Building & Testing

### Prerequisites

- **Android SDK** (API 34+)
- **JDK 17**
- **Node.js** v18+
- **Gradle** (included with the project)

### Build (Quick)

```bash
cd steady-android
./test-build.ps1          # Windows — JS syntax check + APK build
# or
npm install && npx cap sync android && cd android && ./gradlew assembleDebug
```

APK appears at: `steady-android/android/app/build/outputs/apk/debug/app-debug.apk`

### Run Tests

```bash
npm test
```

Tests cover:
- ZIP parsing (Deflate decompression)
- Fitbit data import logic
- POTS threshold calculations

See [**Testing**](DOCUMENTATION.md#testing) for full details.

---

## Contributing

We welcome contributions! Please:

1. **Read [DOCUMENTATION.md](DOCUMENTATION.md#contributing)** for guidelines
2. **Open an issue first** — let's discuss your idea
3. **Fork, edit, test, and submit a PR**

### Code Style

- **JavaScript:** Vanilla ES6, no frameworks, comment the **why** not the **what**
- **Java:** Follow Android conventions, use AndroidX
- **Safety first:** App must stay offline and never send data

### Reporting Bugs or Security Issues

- **Bug?** Open an issue with reproduction steps
- **Security issue?** Don't post publicly — email the maintainer privately
- **Feature request?** Open an issue to discuss

---

## Privacy & Permissions

### What Steady Does

✅ All data stays on your phone  
✅ No cloud sync or backups  
✅ No tracking or analytics  
✅ No accounts or logins  
✅ No ads or third-party scripts  

### Permissions

| Permission | Why | Required? |
|-----------|-----|-----------|
| `READ_EXTERNAL_STORAGE` | Pick Fitbit export files | ✅ (API ≤32) |
| `MANAGE_EXTERNAL_STORAGE` | Auto-scan Downloads for exports | Optional |
| `INTERNET` | **Removed in v1.1** | ❌ No |

**No internet permission means the OS itself blocks any network access.**

See [Privacy & Data Handling](DOCUMENTATION.md#privacy--data-handling) for details.

---

## Roadmap

**v1.1** (Current) — File import fix, chunked reads, native backup

**v1.2** (Planned)
- [ ] Period tracking
- [ ] Medication log
- [ ] CSV export for doctor visits
- [ ] Dark mode (system preference)

**v2.0** (Future)
- [ ] Optional sync across devices (end-to-end encrypted)
- [ ] Wearable integration (direct data, not just exports)
- [ ] Pattern detection (ML-based flare prediction)

---

## FAQ

**Q: Why is there no cloud sync?**  
A: Your health data is sensitive. Cloud sync requires an always-on server (not compatible with a weekend hobby project) and introduces privacy/security risks. Desktop Backup → Phone Restore works fine for syncing across devices and is under *your* control.

**Q: Can I use this on iOS?**  
A: Not currently. Capacitor supports iOS, but the app hasn't been tested on it. Contributions welcome if you want to port it.

**Q: How do I delete my data?**  
A: Uninstall the app (which clears localStorage). Backups are JSON files in your Downloads — delete them manually.

**Q: What if Fitbit changes their export format?**  
A: The ZIP parser is in `steady.html` and can be updated. Open an issue if an export stops working.

**Q: Is my data safe if my phone is stolen?**  
A: Steady data is encrypted by Android's storage encryption, but a thief with physical access might be able to bypass it. **Keep your phone's PIN/biometric lock enabled.** Regular backups (and secure storage of those backups) are your safety net.

---

## License

Steady is released under the **MIT License**. See [LICENSE](LICENSE) for details.

You are free to use, modify, and distribute Steady for any purpose, as long as you include the original copyright notice.

### Third-Party Libraries

Steady uses open-source libraries licensed under MIT and Apache 2.0. See [License & Attribution](DOCUMENTATION.md#license--attribution) for a complete list.

---

## Support & Questions

- **User questions?** See [How to Use Steady](DOCUMENTATION.md#how-to-use-steady)
- **Build problems?** Check [HANDOFF.md](HANDOFF.md)
- **Want to contribute?** Read [Contributing](DOCUMENTATION.md#contributing)
- **Report a bug?** Open a GitHub issue
- **Medical questions?** Consult a healthcare provider

---

## Acknowledgments

Built with care for people managing chronic illness. Special thanks to the EDS and POTS communities for inspiration and feedback.

If you use or fork Steady, we appreciate a mention in your project docs.

---

## Disclaimer

**This app is provided "as-is" for personal health tracking only.** It is not a medical device, does not provide medical advice, and should never be used as a substitute for professional medical care.

**Always consult a healthcare provider** for health concerns, diagnosis, or treatment.

See [Full Legal Disclaimer](DOCUMENTATION.md#full-legal-disclaimer) for complete terms.

---

**Version:** 1.1  
**Last Updated:** 2026-07-04  
**Maintained by:** [Your Name / Organization]  
**GitHub:** https://github.com/your-org/steady
