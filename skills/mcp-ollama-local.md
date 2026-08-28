# Ollama Local LLM Runner MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Ollama Local LLM Runner MCP Server |
| **Category** | AI/Local |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/ollama/mcp-ollama |
| **Transport** | stdio |
| **Install** | `npx -y @ollama/mcp-server` |

## Tools & Capabilities
- `list_local_models` — Query downloaded models (Llama 3.3, Mistral, Qwen 2.5, DeepSeek-R1)
- `generate_chat` — Execute 100% offline local inference on GPU/Metal without external network
- `pull_model` — Download open-weight model tag from Ollama library directly
- `generate_embeddings` — Create offline vector embeddings using `nomic-embed-text` or `bge-m3`
- `show_model_info` — Inspect model architecture, quantization level (Q4_K_M, Q8), and context size

## Client Configuration
```json
{
  "mcpServers": {
    "ollama": {
      "command": "npx",
      "args": [
        "-y",
        "@ollama/mcp-server"
      ],
      "env": {
        "OLLAMA_HOST": "http://127.0.0.1:11434"
      }
    }
  }
}
```

## Security & Best Practices
- Complete local isolation. Zero external network egress or telemetry.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
