# Dify.ai LLM Application MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Dify.ai LLM Application MCP Server |
| **Category** | AI/Agent-Platform |
| **Official** | ⭐ Official (Dify.ai) |
| **Source** | https://github.com/langgenius/mcp-dify |
| **Transport** | stdio |
| **Install** | `npx -y @dify/mcp-server` |

## Tools & Capabilities
- `run_workflow` — Execute Dify visual orchestration workflow with input variables
- `chat_with_app` — Send conversation message to Dify Agent / Chatbot app
- `query_knowledge_base` — Search Dify segmented datasets using hybrid retrieval
- `upload_document_to_dataset` — Ingest local files into Dify knowledge vector index
- `get_app_parameters` — Retrieve defined prompt templates, model configs, and variables

## Client Configuration
```json
{
  "mcpServers": {
    "dify": {
      "command": "npx",
      "args": [
        "-y",
        "@dify/mcp-server"
      ],
      "env": {
        "DIFY_BASE_URL": "https://api.dify.ai/v1",
        "DIFY_API_KEY": "app-..."
      }
    }
  }
}
```

## Security & Best Practices
- Enforce App-level API key scoping. Sanitize uploaded knowledge base documents.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
