# Contributing to Steady

Thank you for your interest in contributing to Steady! This guide explains how to participate.

## Code of Conduct

- Be respectful and supportive
- No harassment, discrimination, or abuse
- This is a health app for vulnerable people — assume good intent and be kind

## Before You Start

**Please open an issue first** to discuss your idea. We want to make sure:
- The feature aligns with Steady's goals (simple, offline, private)
- You're not duplicating work
- We agree on the approach

Examples of good contributions:
- ✅ Bug fixes
- ✅ New symptoms/triggers in the logging interface
- ✅ Improved charts or data visualization
- ✅ Better documentation or guides
- ✅ Security improvements
- ✅ Performance optimizations

Examples of out-of-scope:
- ❌ Cloud sync features (contradicts "offline-first" design)
- ❌ AI training on user data (privacy concern)
- ❌ Ads or monetization
- ❌ Medical advice or diagnosis logic (not our role)

## How to Contribute

### 1. Fork & Clone

```bash
git clone https://github.com/your-fork/steady.git
cd steady
git checkout -b feature/your-feature
```

### 2. Set Up Your Environment

See [HANDOFF.md](HANDOFF.md) for toolchain setup (JDK 17, Android SDK, Node.js).

### 3. Make Changes

**For the app itself:**
- Edit `steady.html` (this is the source of truth)
- No frameworks, no build step
- Keep it simple and vanilla ES6

**For the Android project:**
- Changes to `steady-android/` are auto-generated (except plugins)
- Custom plugin code goes in `android/app/src/main/java/com/sara/steady/`

**For documentation:**
- Update `DOCUMENTATION.md`, `README.md`, or `HANDOFF.md` as needed
- Keep it clear and beginner-friendly

### 4. Test Locally

**JavaScript syntax check:**
```bash
node --check steady.html
```

**Full build:**
```bash
cd steady-android
./test-build.ps1              # Windows
# or
npm install && npx cap sync android && cd android && ./gradlew assembleDebug
```

**Manual testing on device:**
```bash
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
```

Test the affected feature thoroughly. See [Testing](DOCUMENTATION.md#testing) for the checklist.

### 5. Commit with Clear Messages

```bash
git add steady.html DOCUMENTATION.md  # specific files
git commit -m "Add heart rate unit toggle in trends"
```

Commit messages should:
- Start with a verb: "Add", "Fix", "Improve", "Refactor"
- Be clear and specific (not "update stuff")
- Reference the issue: "Fixes #123"

**Bad:** "fix bug"  
**Good:** "Fix POTS threshold alert off by 1 bpm"

### 6. Submit a Pull Request

Push your branch and open a PR on GitHub:
```bash
git push origin feature/your-feature
```

In the PR description:
- **What:** Briefly describe the change
- **Why:** Explain the problem it solves
- **How:** Describe your approach
- **Testing:** List what you tested

Example:
```
## What
Add heart rate unit toggle in Trends (toggle between bpm and HRV)

## Why
Users want to see HRV alongside resting HR to spot autonomic patterns

## How
Added a switch in the Trends header; clicking it re-renders charts with HRV

## Testing
- [x] Manual test: toggle works, charts update
- [x] Syntax check passes
- [x] Build succeeds
- [x] No network calls (checked with adb logcat)
- [x] Data persists across app restart
```

---

## Code Style

### JavaScript (steady.html)

- **No frameworks, no transpilation**
- Vanilla ES6 (for/const/let/arrow functions are fine)
- **Comments explain the "why", not the "what"**
  ```javascript
  // Bad comment
  x = state.feelings.length;  // get number of feelings
  
  // Good comment
  // Count only feelings from the last 30 days for trend relevance
  const recentFeelings = state.feelings.filter(f => daysSince(f.date) <= 30);
  ```

- **Keep functions small and focused**
  ```javascript
  // Bad: does multiple things
  function saveAndRender() { save(); render(); updateCharts(); }
  
  // Good: single responsibility
  function save() { /* ... */ }
  function renderToday() { /* ... */ }
  function updateTrends() { /* ... */ }
  ```

- **Use clear variable names**
  ```javascript
  // Bad
  const x = data.filter(d => d.value > 30);
  
  // Good
  const elevatedReadings = standTests.filter(t => t.resting > 30);
  ```

### Java (FilePickerPlugin.java)

- Follow [Android code style guidelines](https://google.github.io/styleguide/javaguide.html)
- Use AndroidX libraries (no support library)
- All public methods should have `@PluginMethod` annotation
- Catch exceptions; never let them crash the app
  ```java
  // Good: catch and log
  try {
    // ... file operation ...
  } catch (Exception e) {
    Log.e("FilePickerPlugin", "Failed to read file: " + e.getMessage());
    call.reject("Failed to read file");
  }
  ```

### Documentation

- **Clear and beginner-friendly**
- Include examples
- Explain the "why", not just the "what"
- Keep API docs up-to-date

---

## Security & Privacy

### Before Submitting

- [ ] **No network calls.** Verify with `adb logcat` or Charles Proxy that the app makes zero external requests
- [ ] **No user tracking.** No analytics, crash reporting, or identifiable logging
- [ ] **No sensitive data in logs.** Never log passwords, health data, or PII
- [ ] **Validate external input.** If you import data, validate format and bounds
- [ ] **Handle permissions carefully.** Only request what you need; explain why

### Reporting Security Issues

**Do not** post security vulnerabilities in public issues. Email the maintainer privately with:
- What the vulnerability is
- How to reproduce it
- Suggested fix (optional)

We'll work with you to fix it and credit you in the release notes.

---

## Documentation

### Update These Files When...

| File | When to update |
|------|--------|
| `steady.html` | You change any app code |
| `DOCUMENTATION.md` | User-facing features change |
| `README.md` | Project goals, features, or setup change |
| `HANDOFF.md` | Build process or toolchain changes |
| `CONTRIBUTING.md` | Contribution guidelines change |

### Build Documentation

The docs build themselves from markdown. To contribute docs:

```bash
# Edit DOCUMENTATION.md
# Submit a PR
# GitHub will render it automatically
```

For larger doc changes, consider opening an issue first.

---

## Testing Checklist

Before submitting your PR, run through this:

### Code Quality
- [ ] `node --check steady.html` passes (no syntax errors)
- [ ] Android build succeeds: `gradlew assembleDebug`
- [ ] No duplicate method names in Java files
- [ ] No obvious bugs (read your own code carefully)

### Functionality
- [ ] Your change works as expected on a real device (or emulator)
- [ ] No regressions in other features
- [ ] Edge cases handled (empty lists, network loss, etc.)

### Privacy & Security
- [ ] App makes zero network calls
- [ ] No sensitive data in logs
- [ ] Permissions are requested only when needed
- [ ] Imported data is validated

### Documentation
- [ ] Code comments explain non-obvious logic
- [ ] User-facing changes are documented in DOCUMENTATION.md
- [ ] Build/install instructions are still accurate
- [ ] No broken links in markdown files

---

## Common Workflows

### Updating the App

1. **Edit steady.html**
   ```bash
   vim steady.html
   node --check steady.html  # syntax check
   ```

2. **Build & test**
   ```bash
   cd steady-android
   ./test-build.ps1
   adb install -r android/app/build/outputs/apk/debug/app-debug.apk
   ```

3. **Test on device** — manually exercise your change

4. **Commit & push**
   ```bash
   git add steady.html
   git commit -m "Fix trend chart date labels"
   git push origin feature/fix-dates
   ```

### Adding a New Plugin Method

1. **Add method to FilePickerPlugin.java**
   ```java
   @PluginMethod
   public void myNewMethod(PluginCall call) {
     // ... implementation ...
     call.resolve(result);
   }
   ```

2. **Call it from steady.html**
   ```javascript
   const FP = window.Capacitor?.Plugins?.FilePickerPlugin;
   const result = await FP?.myNewMethod();
   ```

3. **Build, test, document**

### Updating Documentation

1. **Edit the relevant markdown file** (DOCUMENTATION.md, README.md, etc.)
2. **Check links** — make sure `[Link](path)` works
3. **Commit**
   ```bash
   git add DOCUMENTATION.md
   git commit -m "Docs: clarify POTS stand test instructions"
   ```

---

## Getting Help

- **Questions about contributing?** Open an issue
- **Need to discuss an idea?** Open an issue
- **Build problems?** Check [HANDOFF.md](HANDOFF.md) first
- **How do I...?** Ask in your PR or open an issue

We're here to help. No question is too basic.

---

## Recognition

We recognize all contributors in:
- Git commit history
- Release notes
- GitHub's contributors graph

If you've contributed code, you'll automatically appear. Thank you!

---

## License

By contributing to Steady, you agree that your contributions will be licensed under the MIT License (see [LICENSE](LICENSE)).

---

**Last updated:** 2026-07-04  
**Version:** 1.0

Thank you for helping make Steady better! ❤️
