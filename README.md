# Steady

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**A free, private symptom tracker for POTS, dysautonomia, and EDS.**
Everything stays on your device. No account. No ads. No internet required.

> ⚠️ **Steady is not a medical device and does not give medical advice.**
> It's a personal tracking tool. Stand test results are for your own reference —
> always talk to a healthcare provider about diagnosis and treatment.
> [Full legal disclaimer](DOCUMENTATION.md#full-legal-disclaimer)

## ▶ Start using Steady (easiest)

**[Open Steady in your browser](https://grayskl.github.io/Steady/)** — works on any phone or computer.

To make it feel like a real app on your phone:

- **Android (Chrome):** open the link → tap the ⋮ menu → **"Add to Home screen"** → Add.
- **iPhone (Safari):** open the link → tap the Share button (square with arrow) → **"Add to Home Screen"** → Add.

That's it. Steady now has its own icon and works even with no internet.

## ⬇ Or install the Android app

1. On your Android phone (Android 10 or newer), tap:
   **[Download Steady for Android](https://github.com/GraySKL/Steady/releases/latest/download/steady.apk)**
2. Open the downloaded file. Your phone may warn you about installing apps from
   outside the Play Store — this is normal for apps distributed on GitHub.
   Tap **Settings → Allow from this source**, then go back and tap **Install**.
3. Open Steady and start logging.

## What Steady does

- **POTS stand test** — guided 10-minute test with heart-rate tracking
- **Symptom & trigger logging** — quick tap-to-log, designed for low-energy days
- **Daily mood check-in** and **trends** — spot patterns between good and rough days
- **Your own reminders** — meds, meals, water: add times that fit your day, check them off as you go
- **Fitbit import** — read your existing data (optional)
- **Backup & restore** — your data as a file you control

## Your privacy

Steady makes **zero network calls**. The Android app doesn't even have internet
permission — the operating system itself blocks it from going online. No accounts,
no analytics, no cloud. Backups are plain files on your device; treat them like
medical records.

## Good to know

- The **browser version and the Android app store data separately.** To move your
  history from one to the other, use **Backup** in one and **Restore** in the other.
- Your data lives on your device only — do a backup now and then, especially
  before switching phones.

## Questions or problems?

- **How do I use it?** → [User guide](DOCUMENTATION.md#how-to-use-steady)
- **Something's broken?** → [Open an issue](https://github.com/GraySKL/Steady/issues) —
  plain English is fine, no technical detail needed.
- **Medical questions?** → Please ask a healthcare provider, not this project.

---

## For developers

Steady is a single-file vanilla-JS web app (`steady.html`) wrapped in Capacitor
for Android, released under the [MIT License](LICENSE). Contributions welcome —
the app must stay offline-first and privacy-first.

- Architecture, data model, and contributing guide: [DOCUMENTATION.md](DOCUMENTATION.md)
- Build setup and troubleshooting: [BUILDING.md](BUILDING.md) and [HANDOFF.md](HANDOFF.md)
- Security issue? Please report privately via
  [GitHub's private vulnerability reporting](https://github.com/GraySKL/Steady/security/advisories/new)
  rather than a public issue.

Built with care for the EDS and POTS communities.
**Maintained by GraySKL** · Version 2.1
