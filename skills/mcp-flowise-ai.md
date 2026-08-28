# Flowise Drag & Drop AI MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Flowise Drag & Drop AI MCP Server |
| **Category** | AI/Agent-Platform |
| **Official** | ⭐ Official (FlowiseAI) |
| **Source** | https://github.com/FlowiseAI/mcp-flowise |
| **Transport** | stdio |
| **Install** | `npx -y @flowise/mcp-server` |

## Tools & Capabilities
- `predict_chatflow` — Trigger visual LangChain/LlamaIndex chatflow prediction
- `list_chatflows` — List deployed visual AI agent flows, memory types, and connected tools
- `upsert_vector_store` — Ingest document chunks into connected vector store nodes
- `get_chat_history` — Retrieve past session turns and memory context from chatflow
- `execute_custom_tool` — Invoke specialized tool node within Flowise execution context

## Client Configuration
```json
{
  "mcpServers": {
    "flowise": {
      "command": "npx",
      "args": [
        "-y",
        "@flowise/mcp-server"
      ],
      "env": {
        "FLOWISE_BASE_URL": "http://localhost:3000",
        "FLOWISE_API_KEY": "YOUR_FLOWISE_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce API key authentication. Protect custom JavaScript tool execution nodes.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
