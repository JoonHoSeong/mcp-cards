# Replicate Models MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | replicate-mcp |
| **Category** | AI/ML |
| **Official** | Yes |
| **Source** | https://replicate.com/docs/reference/mcp |
| **Transport** | stdio |
| **Install** | `npx @replicate/mcp-server` |

## Description
Replicate MCP Server enables AI agents to run, manage, and monitor machine learning models hosted on Replicate's cloud platform. It provides access to thousands of open-source models including image generation, language models, and audio processing — allowing LLM workflows to orchestrate model inference, track predictions, and explore available models directly from the IDE.

## Key Tools
| Tool | Description |
|------|-------------|
| `run_model` | Run a model with specified inputs and parameters |
| `get_prediction` | Get the status and output of a prediction |
| `list_models` | List available models in your account |
| `search_models` | Search the Replicate model catalog |
| `create_prediction` | Create an async prediction for long-running models |
| `get_model_versions` | List available versions of a model |
| `cancel_prediction` | Cancel a running prediction |

## Configuration
```json
{
  "mcpServers": {
    "replicate": {
      "command": "npx",
      "args": ["@replicate/mcp-server"],
      "env": {
        "REPLICATE_API_TOKEN": "r8_your_token_here"
      }
    }
  }
}
```

## Use Cases
1. Run image generation models (SDXL, Flux) and retrieve outputs within development workflows
2. Compare outputs across different model versions for quality evaluation
3. Orchestrate multi-model pipelines combining vision, language, and audio models

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Run image models | physical-ai-defect-image-generation | Generate synthetic defect images via Replicate models for training data |
| Deploy custom models | nemotron-customize | Deploy and test customized Nemotron models on Replicate infrastructure |

## Prerequisites
- `REPLICATE_API_TOKEN` from https://replicate.com/account/api-tokens
- Node.js 18+ (for npx)

## Security Notes
- API tokens grant full access to your Replicate account including billing
- Monitor usage to prevent unexpected costs from large-scale model inference

## References
- https://replicate.com/docs/reference/mcp
- https://replicate.com/docs/get-started
