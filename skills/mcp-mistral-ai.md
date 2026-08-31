# Mistral AI Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Mistral AI Platform MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (Mistral AI) |
| **Source** | https://github.com/mistralai/mcp-mistral |
| **Transport** | stdio |
| **Install** | `npx -y @mistralai/mcp-server` |

## Tools & Capabilities
- `codestral_fim` — Execute Fill-in-the-Middle (FIM) code completion and surgical refactoring
- `chat_completion` — Generate responses with Mistral Large, Pixtral Large, and Ministral
- `ocr_document` — Extract structured text and tables from PDF and scanned documents with Mistral OCR
- `generate_embeddings` — Produce dense vector representations with `mistral-embed`

## Client Configuration
```json
{
  "mcpServers": {
    "mistral": {
      "command": "npx",
      "args": [
        "-y",
        "@mistralai/mcp-server"
      ],
      "env": {
        "MISTRAL_API_KEY": "YOUR_MISTRAL_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- API Key authentication. Enforce prompt parameter limits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
