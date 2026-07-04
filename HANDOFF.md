# Steady — Standalone Android App: Handoff

**Last updated:** 2026-07-04 (fix round 2)
**Goal:** Turn the Steady health app into a real installed Android app that lives entirely on Sara's phone — works offline, no link, not connected to anything external. Plus a **"Find my download"** button that auto-imports the newest `takeout-*.zip` from the phone's Downloads.

---

## Where we are (status)

**Done ✅**
- Java (JDK 17), Android SDK, and all build tools installed
- Capacitor app created, full Steady app bundled inside
- App v1.0 built, installed, and running on Sara's phone
- Git repos set up for both source and Android project
- Build smoke test script (`test-build.ps1`): JS syntax check + duplicate-method check + build
- **v1.1 APK built (2026-07-04) with the import fix — awaiting install + test on the phone**

**Fixed 🔧 (2026-07-04, v1.1) — the import saga, actual root causes:**
The earlier diagnosis ("WebView doesn't support file inputs") was wrong. Three real bugs:
1. **`FilePickerPlugin` was never registered.** It was missing the `@CapacitorPlugin(name=...)` annotation (Capacitor 6 logs an error and silently skips such plugins) and was registered in `onStart()` — too late; Capacitor injects plugin JS proxies at page load. Now: annotated, and registered in `MainActivity.onCreate()` **before** `super.onCreate()`. Also switched to Capacitor 6's proper `startActivityForResult(call, intent, "pickFileResult")` + `@ActivityCallback` instead of the static `onActivityResult` hack.
2. **The last edit round left a JS syntax error** in `steady.html` (a `${isNative?'...'}` ternary missing its `:''` branch at the import-label line) — the whole script failed to parse, so *every* button died. That's what "buttons stopped working entirely" was. `test-build.ps1` now extracts the `<script>` and runs `node --check` so this can never ship again.
3. **That same edit deleted the `#doBackup`/`#doRestore` click wiring** — restored.

Plus hardening while in there:
- **Chunked file reads** (`readFileChunk`, 4 MB per bridge hop): Sara's takeout zip is 119 MB; one giant base64 string through the Capacitor bridge would OOM/fail. `pickAndImport`, `restoreNative`, and `findDownload` all use it.
- **Native Backup**: blob-anchor downloads do nothing in a WebView, so on-device backups now save via `saveToDownloads` (MediaStore.Downloads — needs **no permission** on Android 10+). Web version keeps the blob download.
- **Native Restore**: uses the same native picker (system file dialog needs no storage permission).
- **`INTERNET` permission removed** from the manifest — OS-level proof the app can't phone home.
- `versionCode` 2 / `versionName` 1.1. Fixed `FP` out-of-scope ReferenceError in `showPermissionBanner`.

**Remaining ⏳**
1. Install v1.1 APK on Sara's phone, test: Import Fitbit data (picker), Backup (should land in Downloads), Restore
2. "Find my download" still gated on the All-files-access permission (greyed out on her phone — device restriction). The picker path doesn't need it, so this is now a nice-to-have.

---

## Key locations

| What | Path |
|---|---|
| **Source app (single source of truth)** | `C:\Users\saral\Documents\Health App\steady.html` |
| **Android build project** | `C:\Users\saral\steady-android` |
| **APK appears at** | `C:\Users\saral\steady-android\android\app\build\outputs\apk\debug\app-debug.apk` |
| Java (JDK 17) | `C:\Users\saral\android-dev\jdk\jdk-17.0.19+10` |
| Android SDK | `C:\Users\saral\android-dev\android-sdk` |

**Git repos:**
- Source: `C:\Users\saral\Documents\Health App` (branch: `master`)
- Android: `C:\Users\saral\steady-android` (branch: `main`)

**App details:** name `Steady`, id `com.sara.steady`, Capacitor 6, `@capacitor/filesystem` 6.0.4.

---

## To rebuild (technical)

Quick rebuild with smoke test:
```powershell
cd C:\Users\saral\steady-android
.\test-build.ps1
```

Full manual build in PowerShell:
```powershell
$dev = "C:\Users\saral\android-dev"
$env:JAVA_HOME = "$dev\jdk\jdk-17.0.19+10"
$env:ANDROID_SDK_ROOT = "$dev\android-sdk"
$env:ANDROID_HOME = "$dev\android-sdk"
Copy-Item "C:\Users\saral\Documents\Health App\steady.html" "C:\Users\saral\steady-android\www\index.html" -Force
Set-Location "C:\Users\saral\steady-android"
npx --yes cap sync android
Set-Location android
& ".\gradlew.bat" assembleDebug --no-daemon
```

---

## To install on the phone (for Sara)

1. Get **`app-debug.apk`** onto your phone — upload to **Google Drive**, then open on phone.
2. Tap the file → **allow installing unknown apps** → install.
3. **Force-close the app** (swipe away from recents) after installing, then reopen.

---

## Known issues

### "Find my download" requires MANAGE_EXTERNAL_STORAGE
The `MANAGE_EXTERNAL_STORAGE` permission is in the manifest but **greyed out** on Sara's phone — the device manufacturer likely blocks this permission. The main **Import Fitbit data** button uses the system file picker instead, which needs no storage permission at all.

### (Resolved 2026-07-04) File import on Android
Was blocking; root causes were plugin-registration bugs and a JS syntax error, not a WebView limitation — see *Fixed 🔧* above. Native picker flow: `pickAndImport()` → `FilePickerPlugin.pickFile` (ACTION_GET_CONTENT) → `readFileChunk` loop → existing `doImportFitbit()`.

---

## Notes

- **Privacy:** the app makes **no network calls** (all data stays on-device). The `INTERNET` permission was removed in v1.1, so the OS itself blocks any network access.
- **Data:** app data lives per-device. Sara's 3 years of Fitbit data was imported into **desktop Chrome's** copy. On the phone app, re-import the same Takeout `.zip` (or use Backup → Restore) to load it there.
- **Data size:** 3 years of Fitbit data = ~77 KB stored in localStorage. The app already filters at import time, keeping only resting HR, sleep, HRV, and steps (one value per day each).

---

## The "why" behind key choices

- **Capacitor bundles the HTML inside the APK** (not a URL wrapper / TWA), so the app is truly standalone and offline.
- **Debug APK** (debug-signed) so it can be sideloaded without Play Store / signing-key setup.
- Build paths use **no spaces** (`steady-android`, `android-dev`) because Android/Gradle builds choke on spaces.

---

## Appendix: rebuilding the toolchain from scratch (disaster recovery)

Only needed if `C:\Users\saral\android-dev` or `C:\Users\saral\steady-android` gets deleted. The machine already has **Node** (v26). Run in PowerShell.

**1. Java (JDK 17):**
```powershell
$dev="C:\Users\saral\android-dev"; New-Item -ItemType Directory -Force $dev | Out-Null
Invoke-WebRequest "https://api.adoptium.net/v3/binary/latest/17/ga/windows/x64/jdk/hotspot/normal/eclipse" -OutFile "$dev\jdk17.zip"
Expand-Archive "$dev\jdk17.zip" -DestinationPath "$dev\jdk" -Force
```

**2. Android SDK command-line tools:**
```powershell
$sdk="$dev\android-sdk"; New-Item -ItemType Directory -Force $sdk | Out-Null
Invoke-WebRequest "https://dl.google.com/android/repository/commandlinetools-win-11076708_latest.zip" -OutFile "$dev\cmdtools.zip"
Expand-Archive "$dev\cmdtools.zip" -DestinationPath "$dev\cmdtools" -Force
New-Item -ItemType Directory -Force "$sdk\cmdline-tools" | Out-Null
Move-Item "$dev\cmdtools\cmdline-tools" "$sdk\cmdline-tools\latest" -Force
```

**3. Accept licenses + install packages:**
```powershell
$env:JAVA_HOME="$dev\jdk\jdk-17.0.19+10"
$lic="$sdk\licenses"; New-Item -ItemType Directory -Force $lic | Out-Null
[System.IO.File]::WriteAllText("$lic\android-sdk-license","`n8933bad161af4178b1185d1a37fbf41ea5269c55`nd56f5187479451eabf01fb78af6dfcb131a6481e`n24333f8a63b6825ea9c5514f83c2829b004d1fee`n")
$sm="$sdk\cmdline-tools\latest\bin\sdkmanager.bat"
& $sm --sdk_root=$sdk "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

**4. Recreate the Capacitor project (if `steady-android` is gone):**
```powershell
$proj="C:\Users\saral\steady-android"; New-Item -ItemType Directory -Force $proj | Out-Null
Set-Location $proj
npm init -y
npm install @capacitor/core@6 @capacitor/cli@6 @capacitor/android@6 @capacitor/filesystem@6
New-Item -ItemType Directory -Force "$proj\www" | Out-Null
Copy-Item "C:\Users\saral\Documents\Health App\steady.html" "$proj\www\index.html" -Force
npx --yes cap init "Steady" "com.sara.steady" --web-dir www
npx --yes cap add android
```
Then **re-apply the two manifest edits** (`requestLegacyExternalStorage="true"` on `<application>`, and `READ_EXTERNAL_STORAGE` + `MANAGE_EXTERNAL_STORAGE` permissions), write `android\local.properties` with `sdk.dir=C:/Users/saral/android-dev/android-sdk`, run `npx cap sync android`, then the build block above.

**Installed versions (reference):** JDK Temurin 17.0.19 · build-tools 34.0.0 · platform-tools 37.0.0 · platforms;android-34 · Capacitor 6 · @capacitor/filesystem 6.0.4 · cmdline-tools 12.0.
