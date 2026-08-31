# Hugging Face Spaces & Gradio Apps MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Hugging Face Spaces & Gradio Apps MCP Server |
| **Category** | AI/Hub |
| **Official** | ⭐ Official (Hugging Face) |
| **Source** | https://huggingface.co/mcp |
| **Transport** | stdio |
| **Install** | `npx -y @huggingface/mcp-spaces` |

## Tools & Capabilities
- `call_gradio_space` — Invoke any public or private Hugging Face Gradio Space as a native agent tool
- `search_spaces` — Query trending spaces by task tag, popularity, and model architecture
- `get_space_schema` — Inspect Gradio function signatures, input components, and return types
- `duplicate_space` — Fork and instantiate custom Space hardware runtime

## Client Configuration
```json
{
  "mcpServers": {
    "hf-spaces": {
      "command": "npx",
      "args": [
        "-y",
        "@huggingface/mcp-spaces"
      ],
      "env": {
        "HF_TOKEN": "hf_..."
      }
    }
  }
}
```

## Security & Best Practices
- Scoped Hugging Face User Access Token. Respect Space GPU hardware tier quotas.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
