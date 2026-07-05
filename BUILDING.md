# Building Steady for Android

This guide covers building Steady from source for Android development and contribution.

> **Note:** The Capacitor Android project (`android-build/` / `steady-android/`) is **not
> yet committed to this repository** — the repo currently contains the web app
> (`steady.html`) plus docs. To build the APK yourself, recreate the wrapper project
> with the steps in [HANDOFF.md](HANDOFF.md#appendix-rebuilding-the-toolchain-from-scratch-disaster-recovery),
> or open an issue if you'd like the project committed.

## Quick Start (10 minutes)

```bash
# Clone the repo
git clone https://github.com/GraySKL/Steady.git
cd Steady/android-build

# Run the automated build script (Windows)
.\test-build.ps1

# Or manually (macOS/Linux)
export JAVA_HOME=/path/to/jdk-17
export ANDROID_SDK_ROOT=$HOME/android-sdk
npx --yes cap sync android
cd android
./gradlew assembleDebug --no-daemon
```

APK appears at: `android/app/build/outputs/apk/debug/app-debug.apk`

## Prerequisites

You need:
- **Node.js** v18+
- **JDK 17** (not 11, not 21 — exactly 17)
- **Android SDK** API 34+
- **Gradle** (bundled with the project)

### Installation (One-Time)

#### macOS

```bash
# Install JDK 17
brew install openjdk@17

# Download Android SDK
# From: https://developer.android.com/studio/command-line-tools
# Extract to: ~/android-sdk

# Accept licenses
mkdir -p ~/android-sdk/licenses
echo -e "\n8933bad161af4178b1185d1a37fbf41ea5269c55\n" > ~/android-sdk/licenses/android-sdk-license

# Install build tools
~/android-sdk/cmdline-tools/latest/bin/sdkmanager \
  "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

#### Linux (Ubuntu/Debian)

```bash
# Install JDK 17
sudo apt-get install openjdk-17-jdk

# Download Android SDK (same as macOS)
# Extract to: ~/android-sdk
# Run sdkmanager (same as macOS)
```

#### Windows (PowerShell)

```powershell
# Download JDK 17 from https://adoptium.net
# Install to C:\jdk-17

# Download Android SDK from https://developer.android.com/studio/command-line-tools
# Extract to C:\android-sdk

# Accept licenses
$lic="C:\android-sdk\licenses"
New-Item -ItemType Directory -Force $lic | Out-Null
[System.IO.File]::WriteAllText("$lic\android-sdk-license",
  "`n8933bad161af4178b1185d1a37fbf41ea5269c55`n")

# Install build tools
$sdk="C:\android-sdk"
& "$sdk\cmdline-tools\latest\bin\sdkmanager.bat" --sdk_root=$sdk `
  "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

## Project Structure

```
Steady/
├── steady.html              # The app source (EDIT THIS)
├── DOCUMENTATION.md         # User & developer guide
├── CONTRIBUTING.md          # Contribution guidelines
├── README.md
├── LICENSE
│
├── android-build/           # Everything below here is Android-specific
│   ├── package.json         # Node dependencies
│   ├── capacitor.config.json
│   ├── www/
│   │   └── index.html       # Copy of steady.html (auto-synced)
│   ├── test-build.ps1       # Windows build script
│   │
│   └── android/
│       ├── app/
│       │   ├── src/main/
│       │   │   ├── AndroidManifest.xml
│       │   │   ├── java/com/sara/steady/
│       │   │   │   ├── MainActivity.java
│       │   │   │   └── FilePickerPlugin.java
│       │   │   └── res/
│       │   └── build.gradle
│       ├── build.gradle
│       └── local.properties (you create this)
```

## The Build Pipeline

1. **Edit `steady.html`** (source of truth for the app)
2. **Run `test-build.ps1`** (or `npx cap sync && gradlew assembleDebug`)
   - ✅ Copies steady.html → android-build/www/index.html
   - ✅ Syntax-checks the JavaScript
   - ✅ Syncs with Capacitor
   - ✅ Builds the APK
   - ✅ Checks for duplicate methods
3. **Test on device** (adb install APK or drag to phone)
4. **Commit & push**

## Development Workflow

### Making App Changes

**You only edit `steady.html`** — the Android project is just a wrapper.

```bash
# 1. Edit steady.html (the app)
vim steady.html

# 2. Rebuild
cd android-build
./test-build.ps1          # Windows
# or manually:
npx --yes cap sync android
cd android
./gradlew assembleDebug --no-daemon

# 3. Test on device
adb install -r android/app/build/outputs/apk/debug/app-debug.apk

# 4. Commit both files
cd ../..
git add steady.html
git commit -m "Fix trend chart label alignment"
git push
```

### Making Native Plugin Changes

If you modify `FilePickerPlugin.java`:

```bash
# 1. Edit android-build/android/app/src/main/java/com/sara/steady/FilePickerPlugin.java

# 2. Rebuild
cd android-build
./test-build.ps1

# 3. Test

# 4. Commit
git add android-build/android/app/src/main/java/com/sara/steady/FilePickerPlugin.java
git commit -m "Add readFileChunk method for large file support"
git push
```

## Troubleshooting

### "JAVA_HOME not found" or "Gradle won't build"

**Windows:**
```powershell
$env:JAVA_HOME = "C:\jdk-17"
$env:ANDROID_SDK_ROOT = "C:\android-sdk"
$env:ANDROID_HOME = "C:\android-sdk"
# Then rebuild
```

**macOS/Linux:**
```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)  # macOS
# or: export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64  # Linux
export ANDROID_SDK_ROOT=$HOME/android-sdk
export ANDROID_HOME=$HOME/android-sdk
# Then rebuild
```

### "APK not found" after build

The build likely failed. Check the console for errors. Common issues:
- JAVA_HOME points to wrong version (must be 17, not 11/21)
- SDK not installed (did you run sdkmanager?)
- Old gradle cache (try `cd android && ./gradlew clean`)

### "JavaScript syntax error"

The `test-build.ps1` script runs `node --check` to catch these before building. Fix the error in `steady.html` and retry.

### "Duplicate methods in FilePickerPlugin"

The plugin has two methods with the same name. This won't compile. Check that `@PluginMethod` methods have unique names.

## Testing

### Unit Tests (JavaScript Logic)

```bash
npm test
```

Tests cover:
- ZIP parsing (Fitbit data import)
- Data model calculations
- Date/timezone handling

### Manual Testing on Device

See [Testing](DOCUMENTATION.md#testing) in DOCUMENTATION.md for the full checklist.

## Debugging

### View Android Logs

```bash
adb logcat | grep -E "FilePickerPlugin|Steady|JavaScript"
```

### Check Bridge Messages

Add logging to `steady.html`:

```javascript
console.log("[Steady]", "Event happened", data);
// Then check adb logcat for [Steady] messages
```

### Debug the Native Plugin

In `FilePickerPlugin.java`, use Android logging:

```java
import android.util.Log;

Log.d("FilePickerPlugin", "pickFile called, result=" + uri.toString());
```

## Contributing a Build Feature

Want to add a feature that needs native code?

1. **Open an issue first** — let's discuss the approach
2. **Implement in FilePickerPlugin.java** (or add a new plugin)
3. **Call it from steady.html** via `window.Capacitor.Plugins.YourPlugin.yourMethod()`
4. **Test on a real device**
5. **Submit a PR** with:
   - Java code changes
   - steady.html changes
   - Testing notes
   - Why this needs native code (not doable in JS)

See [CONTRIBUTING.md](CONTRIBUTING.md) for style guidelines.

## Performance Tips

- **Incremental builds:** `./gradlew assembleDebug --no-daemon` (faster second time)
- **Target a specific device:** `adb install -r -s DEVICE_ID APK_PATH`
- **Use an emulator:** Install Android Studio, create an x86_64 emulator (faster than ARM on most PCs)

## Versioning

- **versionCode** (integer, must increase) — used by Google Play
- **versionName** (string) — user-facing version ("1.2.0")

Update both in `android-build/android/app/build.gradle`:

```gradle
versionCode 4
versionName "1.2"
```

## Releasing to Google Play (Future)

When ready to go on the Play Store:

1. Create a signing key (not debug-signed)
2. Update `build.gradle` with signing config
3. Build release APK: `./gradlew assembleRelease`
4. Upload to Play Console

Currently, we're on GitHub Releases (free, no store approval needed).

## Questions?

- **Build errors?** Check the troubleshooting section above
- **Want to contribute?** Read [CONTRIBUTING.md](CONTRIBUTING.md)
- **Architecture question?** See [DOCUMENTATION.md](DOCUMENTATION.md#architecture)
- **Need help?** Open an issue with:
  - Your OS (macOS/Linux/Windows)
  - Error message (full output)
  - What you were trying to do

---

**Last updated:** 2026-07-04  
**Steady version:** 1.2  
**Target Android:** API 34 (Android 14)  
**Build tools:** Gradle, Capacitor 6, Android SDK


---

# Appendix: "Building & Testing" section relocated from the README

*(Moved here 2026-07-05 when the README was rewritten for non-technical users.)*

## Prerequisites

- **Android SDK** (API 34+)
- **JDK 17**
- **Node.js** v18+
- **Gradle** (included with the project)

## Build (Quick)

```bash
cd steady-android
./test-build.ps1          # Windows â€” JS syntax check + APK build
# or
npm install && npx cap sync android && cd android && ./gradlew assembleDebug
```

APK appears at: `steady-android/android/app/build/outputs/apk/debug/app-debug.apk`

## Run Tests

```bash
npm test
```

Tests cover:
- ZIP parsing (Deflate decompression)
- Fitbit data import logic
- POTS threshold calculations

See [**Testing**](DOCUMENTATION.md#testing) for full details.

