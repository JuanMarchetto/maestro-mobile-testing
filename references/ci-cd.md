# CI/CD Integration

## GitHub Actions with Maestro Cloud

Maestro Cloud provides real devices in CI without local simulators. Use the official action:

```yaml
name: Mobile E2E Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  maestro-e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Build Android APK
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build Android APK
        run: |
          cd apps/mobile
          npx expo prebuild --platform android --no-install
          cd android && ./gradlew assembleRelease

      - name: Run Maestro Cloud Tests
        id: maestro
        uses: mobile-dev-inc/action-maestro-cloud@v2
        with:
          api-key: ${{ secrets.MAESTRO_API_KEY }}
          app-file: apps/mobile/android/app/build/outputs/apk/release/app-release.apk
          workspace: .maestro
          include-tags: ci

      # Access results
      # ${{ steps.maestro.outputs.MAESTRO_CLOUD_CONSOLE_URL }}
      # ${{ steps.maestro.outputs.MAESTRO_CLOUD_UPLOAD_STATUS }}
      # ${{ steps.maestro.outputs.MAESTRO_CLOUD_FLOW_RESULTS }}
```

## Tag-Based Flow Filtering

Use tags to control which tests run in CI vs locally:

```yaml
# In your flow file header
appId: com.myapp
tags:
  - ci
  - smoke
---
- launchApp
# ... test steps
```

```bash
# Run only CI-tagged flows locally
maestro test --include-tags ci .maestro/

# Exclude work-in-progress flows
maestro test --exclude-tags wip .maestro/
```

## Local CI with Docker (Android Only)

```dockerfile
FROM openjdk:17-slim

RUN curl -Ls "https://get.maestro.mobile.dev" | bash
ENV PATH="/root/.maestro/bin:${PATH}"

COPY .maestro/ /app/.maestro/
WORKDIR /app

CMD ["maestro", "test", ".maestro/"]
```

**Note:** iOS tests cannot run in Docker (requires macOS). Use Maestro Cloud for iOS in CI.

## Maestro Cloud

[Maestro Cloud](https://cloud.maestro.dev/) runs tests on real devices without local simulator setup.

### Setup

1. Create account at cloud.maestro.dev
2. Generate API key from dashboard
3. Store as `MAESTRO_API_KEY` secret in your CI provider

### Running from CLI

```bash
# Upload and run on Maestro Cloud
maestro cloud --api-key $MAESTRO_API_KEY \
  --app-file ./app-release.apk \
  .maestro/

# With tag filtering
maestro cloud --api-key $MAESTRO_API_KEY \
  --app-file ./app-release.apk \
  --include-tags smoke \
  .maestro/
```

### Key Points

- **iOS testing**: Supported on Maestro Cloud (not on local physical devices)
- **Android testing**: Both local physical devices and Maestro Cloud
- **Results**: Dashboard with video recordings, logs, and screenshots
- **CI outputs**: `MAESTRO_CLOUD_CONSOLE_URL`, `MAESTRO_CLOUD_FLOW_RESULTS`
