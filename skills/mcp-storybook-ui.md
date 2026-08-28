# Storybook MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Storybook MCP Server |
| **Category** | Frontend/Testing |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/storybookjs/mcp-storybook |
| **Transport** | stdio |
| **Install** | `npx -y @storybook/mcp` |

## Tools & Capabilities
- `list_stories` — List all registered UI component stories in the workspace
- `get_story_props` — Inspect component props, args, and variant states
- `render_story` — Render component state and capture visual snapshot
- `test_accessibility` — Run axe-core accessibility audits on rendered components
- `diff_variants` — Compare visual and functional differences across component variants

## Client Configuration
```json
{
  "mcpServers": {
    "storybook": {
      "command": "npx",
      "args": [
        "-y",
        "@storybook/mcp",
        "--port=6006"
      ],
      "env": {
        "STORYBOOK_URL": "http://localhost:6006"
      }
    }
  }
}
```

## Security & Best Practices
- Operates against local Storybook dev server. Does not transmit component source code externally.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
