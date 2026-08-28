# AWS Bedrock Generative AI MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AWS Bedrock Generative AI MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (AWS Labs) |
| **Source** | https://github.com/awslabs/mcp-bedrock |
| **Transport** | stdio |
| **Install** | `npx -y @awslabs/mcp-bedrock` |

## Tools & Capabilities
- `list_foundation_models` — Query available models (Claude 3.5, Llama 3.3, Amazon Titan)
- `invoke_model` — Execute inference request with temperature, top_p, and stop sequences
- `invoke_model_with_guardrail` — Run inference with Amazon Bedrock Guardrails safety filters
- `retrieve_from_knowledge_base` — Query Bedrock Knowledge Bases with vector RAG search
- `create_model_customization_job` — Start fine-tuning or continued pre-training jobs

## Client Configuration
```json
{
  "mcpServers": {
    "aws-bedrock": {
      "command": "npx",
      "args": [
        "-y",
        "@awslabs/mcp-bedrock"
      ],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_ACCESS_KEY_ID": "YOUR_AWS_ACCESS_KEY",
        "AWS_SECRET_ACCESS_KEY": "YOUR_AWS_SECRET_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce Bedrock Guardrails. Log token consumption and latency metrics per request.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
