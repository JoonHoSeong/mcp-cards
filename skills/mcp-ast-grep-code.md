# ast-grep Structural Code Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | ast-grep Structural Code Search MCP Server |
| **Category** | Developer/Search |
| **Official** | ⭐ Official (ast-grep) |
| **Source** | https://github.com/ast-grep/mcp-server-ast-grep |
| **Transport** | stdio |
| **Install** | `npx -y @ast-grep/mcp-server` |

## Tools & Capabilities
- `search_code_pattern` — Search codebase using AST syntax patterns (e.g. `$A.filter($B)` across JS/TS/Python/Rust)
- `rewrite_code_pattern` — Perform syntax-aware structural code replacements across entire repository
- `lint_with_rules` — Execute custom AST yaml lint rules and report violation nodes

## Client Configuration
```json
{
  "mcpServers": {
    "ast-grep": {
      "command": "npx",
      "args": [
        "-y",
        "@ast-grep/mcp-server"
      ]
    }
  }
}
```

## Security & Best Practices
- Local AST tree-sitter parsing. Fast in-memory execution with dry-run support for rewrites.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
