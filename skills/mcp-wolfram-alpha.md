# Wolfram Alpha Computational Knowledge MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Wolfram Alpha Computational Knowledge MCP Server |
| **Category** | Math/Science |
| **Official** | ⭐ Official (Wolfram Alpha) |
| **Source** | https://github.com/wolfram/mcp-wolfram-alpha |
| **Transport** | stdio |
| **Install** | `npx -y @wolfram/mcp-server` |

## Tools & Capabilities
- `query_wolfram_alpha` — Execute symbolic calculus, linear algebra, physics calculations, and unit conversions
- `evaluate_wolfram_language` — Run complex algorithms and scientific models in Wolfram Engine
- `get_step_by_step_solution` — Retrieve detailed algebraic step-by-step solution derivations
- `render_math_plot` — Generate mathematical function plots and 3D surface charts

## Client Configuration
```json
{
  "mcpServers": {
    "wolfram": {
      "command": "npx",
      "args": [
        "-y",
        "@wolfram/mcp-server"
      ],
      "env": {
        "WOLFRAM_APP_ID": "YOUR_WOLFRAM_APP_ID"
      }
    }
  }
}
```

## Security & Best Practices
- Wolfram Full Results API AppID authentication. Stateless query execution.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
