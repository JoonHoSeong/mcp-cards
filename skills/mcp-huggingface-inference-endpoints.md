# Hugging Face Inference Endpoints MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Hugging Face Inference Endpoints MCP Server |
| **Category** | AI/Inference |
| **Official** | ⭐ Official (Hugging Face) |
| **Source** | https://github.com/huggingface/mcp-inference-endpoints |
| **Transport** | stdio |
| **Install** | `npx -y @huggingface/mcp-endpoints` |

## Tools & Capabilities
- `scale_endpoint` — Scale dedicated TGI/TEI inference endpoint replicas up or down
- `deploy_custom_model` — Deploy private fine-tuned model checkpoint to dedicated cloud GPU
- `get_endpoint_status` — Inspect latency, throughput metrics, and endpoint health
- `invoke_endpoint` — Send batched inference requests with custom decoding parameters

## Client Configuration
```json
{
  "mcpServers": {
    "hf-endpoints": {
      "command": "npx",
      "args": [
        "-y",
        "@huggingface/mcp-endpoints"
      ],
      "env": {
        "HF_TOKEN": "hf_..."
      }
    }
  }
}
```

## Security & Best Practices
- Enforce endpoint autoscaling ceilings to control cloud compute expenditure.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
