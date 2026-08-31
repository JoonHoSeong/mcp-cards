# Anthropic Sequential Thinking MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Anthropic Sequential Thinking MCP Server |
| **Category** | AI/Reasoning |
| **Official** | ⭐ Official (Anthropic Reference) |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-sequential-thinking` |

## Tools & Capabilities
- `sequentialthinking` — Dynamic step-by-step reasoning tool enabling iterative problem decomposition, thought revision, hypothesis testing, and branching logic exploration

## Client Configuration
```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

## Security & Best Practices
- Local in-memory execution. Safe computational sandbox with zero external network dependencies or telemetry.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| Multi-Step Agentic Planning | `nemo-rl-auto-research` / `tao-run-automl` | Decompose complex hyperparameter exploration and architecture optimization into verifiable sub-steps |
| Complex Distributed Debugging | `doca-flow-programming` / `nemo-mbridge-resiliency` | Step-by-step failure root-cause analysis and automated remediation planning |
