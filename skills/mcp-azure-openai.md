# Azure OpenAI Service MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Azure OpenAI Service MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (Microsoft) |
| **Source** | https://github.com/azure/mcp-openai |
| **Transport** | stdio |
| **Install** | `npx -y @azure/mcp-openai` |

## Tools & Capabilities
- `list_deployments` — List deployed models (GPT-4o, o1, Text-Embedding-3)
- `chat_completion` — Execute chat completions with enterprise VNet isolation
- `create_embeddings` — Generate dense vector representations for RAG ingestion
- `inspect_content_filter` — Review Azure Content Filtering results and severity flags
- `get_quota_metrics` — Monitor Tokens-Per-Minute (TPM) utilization and rate limits

## Client Configuration
```json
{
  "mcpServers": {
    "azure-openai": {
      "command": "npx",
      "args": [
        "-y",
        "@azure/mcp-openai"
      ],
      "env": {
        "AZURE_OPENAI_ENDPOINT": "https://your-resource.openai.azure.com/",
        "AZURE_OPENAI_API_KEY": "YOUR_AZURE_OPENAI_KEY",
        "AZURE_OPENAI_API_VERSION": "2024-02-15-preview"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce Managed Identity (Entra ID) authentication in corporate production environments.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
