# Ponytail (Lazy Senior Dev) MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Ponytail (Lazy Senior Dev) MCP Server |
| **Category** | Developer/Optimization |
| **Official** | ⭐ Official (DietrichGebert/ponytail) |
| **Source** | https://github.com/DietrichGebert/ponytail |
| **Transport** | stdio |
| **Install** | `npx -y @dietrichgebert/mcp-ponytail` |

## Tools & Capabilities
- `apply_yagni_rules` — Enforce YAGNI (You Ain't Gonna Need It) principles, preventing over-engineering and boilerplate
- `audit_code_simplicity` — Analyze diffs and prune unnecessary abstractions, wrappers, and redundant dependencies
- `suggest_lazy_implementation` — Recommend the most concise, standard-library-first implementation path
- `review_token_efficiency` — Measure token reduction and lines-of-code savings (up to 50%+ reduction)

## Client Configuration
```json
{
  "mcpServers": {
    "ponytail": {
      "command": "npx",
      "args": [
        "-y",
        "@dietrichgebert/mcp-ponytail"
      ]
    }
  }
}
```

## Security & Best Practices
- Local AST and rules engine execution. Zero external data transmission.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
