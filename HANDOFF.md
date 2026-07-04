# Steady — Standalone Android App: Handoff

**Last updated:** 2026-07-04
**Goal:** Turn the Steady health app into a real installed Android app that lives entirely on Sara's phone — works offline, no link, not connected to anything external. Plus a **"Find my download"** button that auto-imports the newest `takeout-*.zip` from the phone's Downloads.

---

## Where we are (status)

**Done ✅**
- Java (JDK 17), Android SDK, and all build tools installed on this computer
- Capacitor app project created, with the full Steady app bundled inside it
- Storage permissions + the native "Find my download" button added
- Project synced and ready to build

**Remaining ⏳**
1. Run the final build to produce the app file (`app-debug.apk`)
2. Copy that file to the phone and install it
3. Test on the phone (import + the Find-my-download button)

> **The build was paused right before step 1.** Everything is saved. Nothing is lost.

---

## Key locations

| What | Path |
|---|---|
| **Source app (single source of truth)** | `C:\Users\saral\Documents\Health App\steady.html` |
| Web/artifact version (same code) | https://claude.ai/code/artifact/ae9a68f5-80e4-449a-891b-759ea11c1b26 |
| **Android build project** | `C:\Users\saral\steady-android` |
| App code bundled into project | `C:\Users\saral\steady-android\www\index.html` |
| Android manifest (permissions) | `C:\Users\saral\steady-android\android\app\src\main\AndroidManifest.xml` |
| **APK will appear at** | `C:\Users\saral\steady-android\android\app\build\outputs\apk\debug\app-debug.apk` |
| Java (JDK 17) | `C:\Users\saral\android-dev\jdk\jdk-17.0.19+10` |
| Android SDK | `C:\Users\saral\android-dev\android-sdk` |

**App details:** name `Steady`, id `com.sara.steady`, Capacitor 6, `@capacitor/filesystem` 6.0.4.

---

## To finish the build (for a future session / technical)

The tool paths are **not** in the system PATH — the build environment must set them each run. Run in PowerShell:

```powershell
$dev = "C:\Users\saral\android-dev"
$env:JAVA_HOME = "$dev\jdk\jdk-17.0.19+10"
$env:ANDROID_SDK_ROOT = "$dev\android-sdk"
$env:ANDROID_HOME = "$dev\android-sdk"
Set-Location "C:\Users\saral\steady-android\android"
& ".\gradlew.bat" assembleDebug --no-daemon
```

- **First build is slow** (downloads Gradle + dependencies, several minutes). Later builds are fast.
- Success = a `BUILD SUCCESSFUL` line and the APK at the path in the table above.
- `local.properties` is already written with the SDK path.

### If the app source changes later
Re-bundle before rebuilding:
```powershell
Copy-Item "C:\Users\saral\Documents\Health App\steady.html" "C:\Users\saral\steady-android\www\index.html" -Force
Set-Location "C:\Users\saral\steady-android"
npx --yes cap sync android
```
Then run the build block above.

---

## To install on the phone (for Sara — one step at a time)

1. Get **`app-debug.apk`** onto your phone — easiest is to upload it to **Google Drive**, then open it on your phone.
2. Tap the file. Android will ask to **allow installing unknown apps** — say yes (it's your own app).
3. Install it. **Steady** appears in your apps like any other. 💚

*(This is a normal, safe step for a personal app — it's just not coming from the Play Store.)*

---

## Notes & honest caveats

- **Not tested on a real phone yet.** The build should work; behaviour on the actual device (especially the download-finding) may need a tweak. Expect maybe one round of fixes.
- **"Find my download" button** (native only — hidden in the web version):
  - It scans the phone's **Downloads** for the newest `takeout-*.zip` and imports it automatically.
  - Android's file rules are strict. On newer Android it may need a **one-time toggle**: Settings → Apps → **Steady** → Permissions → **allow access to all files**.
  - If it can't read Downloads, it **falls back** to the normal file picker ("Or pick the file myself") — so import always works either way.
- **Privacy:** the app makes **no network calls** in its code (all data stays on-device). The `INTERNET` permission is Capacitor's default; once we confirm the app runs, we can remove it to make it provably offline.
- **Data note:** app data lives per-device. Sara's 3 years of Fitbit data was imported into **desktop Chrome's** copy. On the phone app, re-import the same Takeout `.zip` (or use Backup → Restore) to load it there.

---

## The "why" behind key choices
- **Capacitor bundles the HTML inside the APK** (not a URL wrapper / TWA), so the app is truly standalone and offline — matching Sara's "not connected to anything" requirement.
- **Debug APK** (debug-signed) so it can be sideloaded without a Play Store / signing-key setup.
- Build paths use **no spaces** (`steady-android`, `android-dev`) because Android/Gradle builds choke on spaces (the `Documents\Health App` folder has one).

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

**3. Accept licenses (write hash files — the reliable way) + install packages:**
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
Then **re-apply the two manifest edits** (see the *Notes & caveats* section: `requestLegacyExternalStorage="true"` on `<application>`, and the `READ_EXTERNAL_STORAGE` + `MANAGE_EXTERNAL_STORAGE` permissions), write `android\local.properties` with `sdk.dir=C:/Users/saral/android-dev/android-sdk`, run `npx cap sync android`, then the build block at the top.

**Installed versions (reference):** JDK Temurin 17.0.19 · build-tools 34.0.0 · platform-tools 37.0.0 · platforms;android-34 · Capacitor 6 · @capacitor/filesystem 6.0.4 · cmdline-tools 12.0.

> Note: the `Find my download` button lives in the **source** `steady.html` (gated by an `isNative` check) — it is already there, so a rebuild picks it up automatically once `steady.html` is copied into `www`.
