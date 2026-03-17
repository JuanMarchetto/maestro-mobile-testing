# Critical Gotchas

## clearState Does NOT Clear iOS Keychain

`clearState: true` clears the app sandbox (UserDefaults, files, caches) but does **NOT** clear the iOS Keychain. Auth tokens stored via `expo-secure-store` (or any Keychain-based storage) persist across `clearState` resets and even app reinstalls.

```yaml
# WRONG — user may still be authenticated
- launchApp:
    clearState: true
- assertVisible: "Welcome"  # Fails if Keychain has tokens

# CORRECT — wait for auth resolution, then adapt
- launchApp
- extendedWaitUntil:
    visible:
      id: "auth-loaded"
    timeout: 15000
```

**Rules:**
- Never rely on `clearState` to produce guest state on iOS
- For auth tests: skip `clearState`, use `auth-loaded` pre-flight
- For guest tests: use adaptive flows that handle both states
- Never assert guest-only UI after `clearState`

**Note:** On Android, `clearState: true` fully resets app data including credentials. This is an iOS-only gotcha.

## XCTest kAXErrorInvalidUIElement Crash (iOS)

The XCTest driver may crash if Maestro interacts with the accessibility tree before the first render cycle completes on cold boot.

**Fix:** Add a no-op swipe immediately after `launchApp`:

```yaml
- launchApp
- swipe:
    direction: DOWN
    duration: 100
```

## API Server Dependency

Mobile apps calling backend APIs on localhost need either the full server or a mock server running. Without it, all API-dependent screens show loading spinners or empty states (queries fail silently).

**Fix:** Start a mock API server before running Maestro tests:

```bash
# Start mock server (serves canned responses on your API port)
npx tsx scripts/mock-api-server.ts &

# Then run tests
maestro test .maestro/my-test.yaml
```

Create a lightweight mock that returns canned JSON for each endpoint your app calls. This is faster and more deterministic than running your full backend.

## Android-Specific Patterns

### Emulator Setup

Android tests require an emulator or a USB-connected physical device. Maestro auto-detects connected devices.

```bash
# List available system images
sdkmanager --list | grep system-images

# Create emulator
avdmanager create avd -n maestro_test \
  -k "system-images;android-34;google_apis;arm64-v8a"

# Start emulator
emulator -avd maestro_test
```

### iOS vs Android Differences

| Aspect | iOS | Android |
|--------|-----|---------|
| Device type | Simulator only (no physical) | Emulator + physical via ADB |
| `clearState` | Does NOT clear Keychain | Fully resets app data |
| Cold boot crash | XCTest kAXError (add swipe delay) | No equivalent issue |
| Performance | Runs natively (fast) | ARM emulation (slower on x86) |
| Permission dialogs | System alerts | System dialogs with different text |

### ADB Debugging

```bash
adb devices                                    # List connected devices
adb shell am start -n com.myapp/.MainActivity  # Launch app
adb logcat | grep Maestro                      # Filter Maestro logs
adb shell input keyevent 82                    # Unlock screen
```

### Android Permission Handling

Android permissions appear as system dialogs. Dismiss with optional taps:

```yaml
- tapOn:
    text: "Allow"
    optional: true

- tapOn:
    text: "While using the app"
    optional: true
```

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Unable to locate Java Runtime" | Java not in PATH | `export JAVA_HOME=/opt/homebrew/opt/openjdk@17/...` |
| "Element not found" after tap | Native alert blocking | Add `tapOn: text: "OK" optional: true` |
| OTP digits not entering | Auto-focus interference | Use individual `otp-input-N` testIDs |
| Test passes but nothing happened | `optional: true` misused | Only use optional for truly optional actions |
| "Assertion is false" on visibility | Element not rendered yet | Increase timeout or verify testID exists |
| Script output empty | Wrong JS API | Use `http.get()` not `fetch()` |
| Auth state inconsistent after clearState | iOS Keychain not cleared | Don't use `clearState`, use adaptive flows |
| kAXErrorInvalidUIElement crash | Cold boot race (iOS) | Add post-launch swipe delay |
| Loading spinners / empty screens | No API server running | Start mock API server before tests |
| Permission dialog blocking (Android) | System dialog not dismissed | Add `tapOn: text: "Allow" optional: true` |
