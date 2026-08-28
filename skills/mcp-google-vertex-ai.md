# Google Vertex AI MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Google Vertex AI MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (Google Cloud) |
| **Source** | https://github.com/google-cloud/mcp-vertexai |
| **Transport** | stdio |
| **Install** | `npx -y @google-cloud/mcp-vertexai` |

## Tools & Capabilities
- `generate_content` — Call Gemini 1.5 Pro/Flash models with multimodal payloads
- `list_endpoints` — Query deployed Vertex AI custom model prediction endpoints
- `predict_custom_model` — Send real-time inference request to custom endpoint
- `search_vector_search` — Query Vertex AI Vector Search index with embedding vectors
- `run_pipeline_job` — Trigger Kubeflow-based Vertex AI Training/Evaluation pipelines

## Client Configuration
```json
{
  "mcpServers": {
    "vertexai": {
      "command": "npx",
      "args": [
        "-y",
        "@google-cloud/mcp-vertexai"
      ],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/sa.json",
        "VERTEX_PROJECT_ID": "your-project-id",
        "VERTEX_LOCATION": "us-central1"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce Vertex AI safety filters and quota governance per service account.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
