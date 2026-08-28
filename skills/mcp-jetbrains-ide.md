# JetBrains IDEs Gateway MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | JetBrains IDEs Gateway MCP Server |
| **Category** | Developer/IDE |
| **Official** | ⭐ Official (JetBrains) |
| **Source** | https://github.com/JetBrains/mcp-jetbrains |
| **Transport** | stdio |
| **Install** | `npx -y @jetbrains/mcp-server` |

## Tools & Capabilities
- `navigate_to_symbol` — Navigate active IDE editor (IntelliJ, PyCharm, WebStorm) to class/method
- `inspect_inspections` — Run JetBrains code inspections and static analysis linter checks
- `execute_run_configuration` — Trigger local debug/run configurations within IDE process
- `refactor_rename` — Perform IDE-native semantic symbol renaming across entire project
- `get_active_editor_context` — Retrieve cursor position, selected code text, and active file path

## Client Configuration
```json
{
  "mcpServers": {
    "jetbrains": {
      "command": "npx",
      "args": [
        "-y",
        "@jetbrains/mcp-server",
        "--port=63342"
      ]
    }
  }
}
```

## Security & Best Practices
- Local IDE REST Gateway connection via localhost port 63342 with token validation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
