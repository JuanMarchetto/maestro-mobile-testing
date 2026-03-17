# Core Patterns

## 1. Selector Strategy: testID vs Text

Choose your selector approach based on project context. Both are valid — the right choice depends on whether your app is localized and your team's testing philosophy.

| Context | Recommended Selector | Rationale |
|---------|---------------------|-----------|
| **Multi-language / i18n** | `id:` (testID) | Stable across translations |
| **Single language** | Text labels | Human-readable, self-documenting tests |
| **Agent-maintained tests** | Either — ask the developer | Readability matters less for AI-maintained flows |
| **System dialogs** | Text (always) | No testID possible on native alerts |

```yaml
# testID selector — stable across translations
- tapOn:
    id: "submit-button"

# Text selector — human-readable, self-documenting
- tapOn: "Submit"
```

**When to prefer testIDs:**
- App supports multiple languages or will be translated
- UI text is dynamic or frequently changes
- Multiple elements share the same visible text

**When to prefer text selectors:**
- Single-language app with stable copy
- Readability and self-documentation are a priority
- Testing user-visible behavior exactly as it appears

In React Native, add `testID` props when using ID-based selectors:

```tsx
<TouchableOpacity testID="submit-button" onPress={handleSubmit}>
  <Text>{t('submit')}</Text>
</TouchableOpacity>
```

### testID Naming Convention

When using ID-based selectors:

```
{component}-{action/type}[-{variant}]

Examples:
- auth-prompt-login-button
- product-card-{id}
- otp-input-0
- tab-home
- dashboard-loading
```

## 2. Auth Pre-Flight Pattern

Prevent race conditions where Maestro interacts with the UI before auth state resolves. Add a zero-size `auth-loaded` marker that only renders when auth loading completes:

```tsx
// In your tab bar or root layout
{!isLoading && <View testID="auth-loaded" style={{ width: 0, height: 0 }} />}
```

Then in every test:

```yaml
- launchApp

# Prevent XCTest crash on cold boot (iOS)
- swipe:
    direction: DOWN
    duration: 100

# Wait for auth state to resolve
- extendedWaitUntil:
    visible:
      id: "auth-loaded"
    timeout: 15000

# Now safe to interact
- tapOn:
    id: "tab-home"
```

## 3. Adaptive Tests (Handle Both Auth States)

Tests should work regardless of whether the user is authenticated:

```yaml
# Auth flow — only runs if login prompt is visible
- runFlow:
    when:
      visible: "Sign In"
    file: flows/auth-flow.yaml

# Already authenticated — proceed directly
- runFlow:
    when:
      visible:
        id: "tab-home"
    file: flows/authenticated-action.yaml
```

## 4. Testing Optimistic Updates

Use short timeouts to verify UI changes happen before server response:

```yaml
# Trigger mutation
- tapOn:
    id: "action-button"

# OPTIMISTIC: UI must change within 3s (not waiting for server)
- extendedWaitUntil:
    visible:
      id: "undo-button"
    timeout: 3000

# Verify derived UI state
- extendedWaitUntil:
    visible:
      id: "user-indicator"
    timeout: 5000
```

| Action | Expected Change | Timeout |
|--------|----------------|---------|
| Mutation trigger | Button state flips | < 3s |
| List update | Item appears/disappears | < 5s |
| Re-do action | Proves persistence | < 3s |

## 5. Dismissing Native Alerts

React Native `Alert.alert()` creates native dialogs that block the UI:

```yaml
- tapOn:
    id: "action-button"

# Wait for expected state change first
- extendedWaitUntil:
    visible:
      id: "new-state-element"
    timeout: 5000

# Dismiss alert (optional in case it already closed)
- tapOn:
    text: "OK"
    optional: true

# Brief delay for alert animation
- swipe:
    direction: DOWN
    duration: 300
```

## 6. Sub-Flows for Reusability

Break repeated sequences into sub-flow files:

```
.maestro/
├── flows/
│   ├── auth-and-return.yaml
│   ├── complete-purchase.yaml
│   └── verify-result.yaml
├── smoke-test.yaml
└── feature-test.yaml
```

```yaml
# In main test
- runFlow:
    file: flows/auth-and-return.yaml
```

## 7. Deep Links (Expo)

Use the Expo scheme from `app.json`, not the bundle ID:

```yaml
# WRONG
- openLink: "com.myapp://profile/settings"

# CORRECT
- openLink: "myapp://profile/settings"
```

Deep links must be registered in your app's deep link handler. Unregistered routes silently fail.

## 8. Platform-Specific Logic

```yaml
- runFlow:
    when:
      platform: ios
    file: flows/ios-specific.yaml

- runFlow:
    when:
      platform: android
    file: flows/android-specific.yaml
```

## 9. Environment Variables

```yaml
appId: com.myapp
env:
  TEST_EMAIL: maestro-test@example.com
  API_BASE_URL: http://localhost:3000
---
- inputText: ${TEST_EMAIL}
```

## 10. Selector State Properties

Use `enabled`, `selected`, `checked`, and `focused` to target elements by their current state. This is useful for validating interactive element states before or after actions.

```yaml
# Only tap the submit button if it's enabled
- tapOn:
    id: "submit-button"
    enabled: true

# Assert a checkbox is checked
- assertVisible:
    id: "terms-checkbox"
    checked: true

# Wait for an input to be focused
- extendedWaitUntil:
    visible:
      id: "email-input"
      focused: true
    timeout: 3000
```

| Property | Values | Use Case |
|----------|--------|----------|
| `enabled` | `true` / `false` | Buttons that disable during submission or until form is valid |
| `checked` | `true` / `false` | Checkboxes, toggle switches |
| `selected` | `true` / `false` | Tab items, segmented controls |
| `focused` | `true` / `false` | Input fields with auto-focus |

## 11. Relative Position Selectors

Distinguish between similar elements by their spatial relationship to other elements. This is more idiomatic and resilient than index-based selection.

```yaml
# BAD — fragile, breaks if order changes
- tapOn:
    text: "Add to Basket"
    index: 1

# GOOD — contextual, self-documenting
- tapOn:
    text: "Add to Basket"
    below:
      text: "Awesome Shoes"
```

Available relative selectors:

```yaml
# Target element below another
- tapOn:
    text: "Buy Now"
    below: "Product Title"

# Target element that is a child of a parent
- tapOn:
    text: "Delete"
    childOf:
      id: "item-card-42"

# Target a parent that contains a specific child
- tapOn:
    containsChild: "Urgent"

# Target by multiple descendants
- tapOn:
    containsDescendants:
      - id: title_id
        text: "Specific Title"
      - "Another descendant text"

# Horizontal positioning
- tapOn:
    text: "Edit"
    rightOf: "Username"
```

| Selector | Meaning |
|----------|---------|
| `below:` | Element is positioned below the referenced element |
| `above:` | Element is positioned above the referenced element |
| `leftOf:` | Element is to the left of the referenced element |
| `rightOf:` | Element is to the right of the referenced element |
| `childOf:` | Element is a direct child of the referenced parent |
| `containsChild:` | Element contains a direct child matching the reference |
| `containsDescendants:` | Element contains all specified descendant elements |
