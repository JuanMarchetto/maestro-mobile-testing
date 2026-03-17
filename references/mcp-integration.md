# Maestro MCP Server

The [Maestro MCP server](https://docs.maestro.dev/getting-started/maestro-mcp) exposes Maestro's full command set as Model Context Protocol tools, letting AI agents **execute tests and interact with devices directly** — not just write YAML.

## How It Complements This Skill

| | **This Skill** | **Maestro MCP** |
|---|---|---|
| **Role** | Teaches correct patterns | Provides runtime execution |
| **Layer** | Authoring (write good YAML) | Execution (run, tap, assert, screenshot) |
| **Output** | Better test files | Live device interaction |

Use both together: this skill ensures the AI writes correct tests; the MCP lets it run them immediately, see failures, and iterate.

## Setup

The MCP ships with the Maestro CLI — no extra install needed:

```bash
# Verify it's available
maestro mcp
```

**Claude Code** — add to project `.mcp.json` or global settings:

```json
{
  "mcpServers": {
    "maestro": {
      "command": "maestro",
      "args": ["mcp"]
    }
  }
}
```

**Claude Desktop** — add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "maestro": {
      "command": "maestro",
      "args": ["mcp"]
    }
  }
}
```

Also supported on Cursor, Windsurf, VS Code, and JetBrains IDEs. See the [Maestro MCP docs](https://docs.maestro.dev/getting-started/maestro-mcp) for IDE-specific setup.

## Key Capabilities

The MCP server exposes 47 tools organized by category:

| Category | Tools | Examples |
|----------|-------|---------|
| UI Interaction | tap, swipe, scroll, long press | `tapOn`, `scrollUntilVisible` |
| Text Input | type, erase, paste, copy | `inputText`, `eraseText` |
| Assertions | visibility, AI-powered | `assertVisible`, `assertWithAI` |
| App Lifecycle | launch, stop, clear state | `launchApp`, `clearState` |
| Device Control | location, orientation, airplane | `setLocation`, `hideKeyboard` |
| Flow Control | run flows, repeat, eval scripts | `runFlow`, `evalScript` |
| Media | screenshots, recording | `takeScreenshot`, `startRecording` |
| AI-Powered | visual assertions, defect detection | `assertWithAI`, `assertNoDefectsWithAI` |

## Write-Run-Fix Loop

With both this skill and the MCP active, the AI can:

1. **Write** a test YAML using patterns from this skill
2. **Run** it via MCP's `runFlow` / `launchApp` + interaction tools
3. **See** failures via screenshots and assertions
4. **Fix** the YAML and re-run — all in one conversation
