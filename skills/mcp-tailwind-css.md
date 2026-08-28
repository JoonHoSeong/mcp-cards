# Tailwind CSS MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Tailwind CSS MCP Server |
| **Category** | Frontend/Styling |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/tailwindlabs/mcp-tailwindcss |
| **Transport** | stdio |
| **Install** | `npx -y @tailwind/mcp-server` |

## Tools & Capabilities
- `suggest_classes` — Suggest utility classes based on natural language design description
- `validate_classes` — Validate Tailwind CSS class names against active tailwind.config.js
- `convert_css_to_tailwind` — Transform raw CSS rules into Tailwind utility classes
- `generate_theme_tokens` — Query custom color palettes, spacing, and typography tokens
- `audit_purge_safety` — Verify dynamic class generation safety for tree-shaking

## Client Configuration
```json
{
  "mcpServers": {
    "tailwindcss": {
      "command": "npx",
      "args": [
        "-y",
        "@tailwind/mcp-server",
        "--config=./tailwind.config.js"
      ]
    }
  }
}
```

## Security & Best Practices
- Pure local AST and token parsing. Zero network telemetry or code transmission.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
