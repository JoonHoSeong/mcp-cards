# Obsidian Markdown Vault MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Obsidian Markdown Vault MCP Server |
| **Category** | Productivity/Notes |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/obsidianmd/mcp-obsidian |
| **Transport** | stdio |
| **Install** | `npx -y @obsidian/mcp-vault` |

## Tools & Capabilities
- `search_vault` — Search local markdown notes by text, frontmatter tags, and aliases
- `read_note` — Read complete markdown content and parse YAML frontmatter metadata
- `append_to_daily_note` — Append log entries or meeting notes to today's daily note
- `create_note` — Create new markdown note with wiki-links (`[[Note]]`) and tags
- `inspect_backlinks` — Discover forward and backward link graphs between notes

## Client Configuration
```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": [
        "-y",
        "@obsidian/mcp-vault",
        "--vault=/path/to/ObsidianVault"
      ]
    }
  }
}
```

## Security & Best Practices
- Local filesystem access restricted exclusively to the designated Obsidian vault root.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
