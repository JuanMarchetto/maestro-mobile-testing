# Authentication Testing

## Architecture

Testing OTP or magic-link authentication in E2E requires capturing emails programmatically. The general pattern:

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Maestro    │────▶│  Auth        │────▶│  Email Capture  │
│  Test       │     │  Provider    │     │  Service        │
└─────────────┘     └──────────────┘     └─────────────────┘
       │                                        │
       │         ┌──────────────────────────────┘
       │         ▼
       │   ┌─────────────┐
       └──▶│  REST API   │ ─── GET /api/v1/messages
           │  (email)    │ ─── Extract OTP code
           └─────────────┘
```

Common email capture services: [Mailpit](https://github.com/axllent/mailpit), [MailHog](https://github.com/mailhog/MailHog), [Ethereal](https://ethereal.email/).

## OTP Fetch Script

Fetch OTP codes from your email capture service using Maestro's GraalJS runtime:

```javascript
// CRITICAL: Maestro uses GraalJS — NO async/await, NO fetch()
var email = typeof EMAIL !== "undefined" ? EMAIL : "test@example.com";
var emailServiceUrl = typeof EMAIL_SERVICE_URL !== "undefined"
  ? EMAIL_SERVICE_URL : "http://localhost:8025";

var response = http.get(emailServiceUrl + "/api/v1/messages");
if (!response.ok) {
  throw new Error("Failed to fetch emails: " + response.status);
}

var data = json(response.body);
// Find the latest email and extract OTP code
var body = data.messages[0].Content.Body;
var match = body.match(/(\d{6})/);
output.OTP_CODE = match[1];
```

## OTP Input Strategy

OTP components with auto-focus need individual digit entry. Tap each input before typing:

```yaml
# Split OTP into digits via helper script
- runScript:
    file: scripts/split-otp.js
    env:
      OTP_CODE: ${output.OTP_CODE}

# Enter each digit by tapping its input
- tapOn:
    id: "otp-input-0"
- inputText: ${output.OTP_0}

- tapOn:
    id: "otp-input-1"
- inputText: ${output.OTP_1}
# ... repeat for all digits
```

> For provider-specific implementations (Supabase + Mailpit, Firebase Auth, Auth0), create a project-level skill that extends this one.

## GraalJS Script Rules

Maestro uses the GraalJS runtime. These constraints are non-negotiable:

| Feature | Status |
|---------|--------|
| `async/await` | **NOT supported** |
| `fetch()` | **NOT supported** |
| `http.get()`, `http.post()` | Use these instead |
| `json()` | Use to parse response bodies |
| `output.VAR` | Set variables for use in YAML flow |
| `var` declarations | Required (use `var`, not `const`/`let` for safety) |

```javascript
// Script template
var response = http.get("http://localhost:8025/api/endpoint");
if (!response.ok) {
  throw new Error("Request failed: " + response.status);
}
var data = json(response.body);
output.RESULT = data.value;
```

## Auth-Aware Tab Bars

Tab bars that show different tabs for guest vs authenticated users will cause selector failures:

| State | Typical Tabs |
|-------|-------------|
| Guest | home, search, cart, profile |
| Auth | home, feed, create, messages, profile |

Only assert tabs that exist in both states, or use adaptive `when:` conditions.
