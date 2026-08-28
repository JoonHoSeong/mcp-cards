# Qwen / DashScope Multi-Modal MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Qwen / DashScope Multi-Modal MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (Alibaba Qwen) |
| **Source** | https://github.com/modelscope/mcp-qwen |
| **Transport** | stdio |
| **Install** | `npx -y @modelscope/mcp-qwen` |

## Tools & Capabilities
- `generate_text` — Generate responses using Qwen 2.5 72B / Qwen 2.5 Max
- `understand_image` — Multimodal vision analysis using Qwen 2.5 VL with bounding box grounding
- `generate_code` — High-precision code generation using Qwen 2.5 Coder 32B
- `generate_embeddings` — Produce dense vector representations for multilingual RAG
- `call_audio_chat` — Real-time speech-to-speech audio interaction using Qwen Omni

## Client Configuration
```json
{
  "mcpServers": {
    "qwen": {
      "command": "npx",
      "args": [
        "-y",
        "@modelscope/mcp-qwen"
      ],
      "env": {
        "DASHSCOPE_API_KEY": "sk-..."
      }
    }
  }
}
```

## Security & Best Practices
- Validate multimodal image URL reachability. Enforce maximum token output limits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
