# Android ADB Mobile Device MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Android ADB Mobile Device MCP Server |
| **Category** | Mobile/Android |
| **Official** | ⭐ Verified (Android Open Source) |
| **Source** | https://github.com/mcp-servers/mcp-adb |
| **Transport** | stdio |
| **Install** | `npx -y @mcp-servers/mcp-adb` |

## Tools & Capabilities
- `capture_screenshot` — Capture device screen and return base64 / PNG image for visual analysis
- `dump_ui_hierarchy` — Inspect Android view hierarchy and XML layout nodes for automated UI testing
- `install_apk` — Install APK build artifact onto connected emulator or physical device
- `execute_shell_command` — Run `adb shell` commands (e.g. `am start`, `pm list packages`, `input tap`)

## Client Configuration
```json
{
  "mcpServers": {
    "adb": {
      "command": "npx",
      "args": [
        "-y",
        "@mcp-servers/mcp-adb"
      ]
    }
  }
}
```

## Security & Best Practices
- Requires ADB authorization on target device. Restrict dangerous root shell operations.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
