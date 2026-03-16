# Maestro Mobile Testing

Comprehensive Maestro E2E testing patterns for React Native/Expo apps.

Covers 11 core patterns, iOS/Android gotchas, OTP authentication testing, CI/CD with GitHub Actions + Maestro Cloud, and MCP server integration.

## Install

```
/plugin marketplace add JuanMarchetto/agent-skills-marketplace
/plugin install maestro-mobile-testing@agent-skills-marketplace
```

Or manually:
```bash
git clone https://github.com/JuanMarchetto/maestro-mobile-testing.git
cp -r maestro-mobile-testing ~/.claude/skills/maestro-mobile-testing
```

## What's Inside

- **11 core patterns**: testID selectors, auth pre-flight, adaptive flows, optimistic updates, native alerts, sub-flows, deep links, platform-specific logic, env vars, state properties, relative selectors
- **Authentication testing**: OTP fetch via GraalJS, email capture services, digit-by-digit input
- **Critical gotchas**: iOS Keychain persistence, XCTest crashes, API server dependency, auth-aware tab bars
- **CI/CD**: GitHub Actions + Maestro Cloud, tag-based flow filtering, Docker for Android
- **MCP integration**: Write-run-fix loops with Maestro MCP server

## Requirements

- [Maestro CLI](https://maestro.dev/) + Java 17+
- iOS Simulator or Android emulator
- React Native / Expo project

## License

[MIT](LICENSE)
