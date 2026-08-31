# Together AI Inference & Fine-Tuning MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Together AI Inference & Fine-Tuning MCP Server |
| **Category** | AI/Inference |
| **Official** | ⭐ Official (Together AI) |
| **Source** | https://github.com/togethercomputer/mcp-together |
| **Transport** | stdio |
| **Install** | `npx -y @togethercomputer/mcp-server` |

## Tools & Capabilities
- `generate_image_flux` — Ultra-fast photorealistic text-to-image synthesis using FLUX.1 Schnell/Dev
- `chat_completion` — High-throughput open model inference (Llama 3.3, DeepSeek, Qwen 2.5)
- `list_dedicated_endpoints` — Query dedicated enterprise model clusters and fine-tuned checkpoints
- `submit_fine_tuning_job` — Dispatch LoRA/Full parameter fine-tuning jobs on custom JSONL datasets

## Client Configuration
```json
{
  "mcpServers": {
    "together": {
      "command": "npx",
      "args": [
        "-y",
        "@togethercomputer/mcp-server"
      ],
      "env": {
        "TOGETHER_API_KEY": "YOUR_TOGETHER_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Token masking and API key encryption in memory.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
