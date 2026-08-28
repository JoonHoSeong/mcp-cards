# HuggingFace Hub MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | hf-mcp-server |
| **Category** | AI/ML |
| **Official** | Yes |
| **Source** | https://github.com/mcp/huggingface/hf-mcp-server |
| **Transport** | stdio |
| **Install** | `npx @huggingface/mcp-server` |

## Description
HuggingFace Hub MCP Server provides AI agents with access to the HuggingFace ecosystem — searching models and datasets, reading model cards, running inference, and exploring Spaces. It connects LLM workflows to the largest open-source ML community, enabling discovery of pre-trained models, evaluation of dataset suitability, and direct model inference through the standardized MCP interface.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_models` | Search HuggingFace model hub with filters |
| `get_model_info` | Get detailed metadata for a specific model |
| `search_datasets` | Search available datasets by task or domain |
| `get_dataset_info` | Get dataset card, size, and schema information |
| `list_spaces` | Browse HuggingFace Spaces applications |
| `run_inference` | Run inference on a hosted model via Inference API |
| `get_model_card` | Retrieve the full model card documentation |

## Configuration
```json
{
  "mcpServers": {
    "huggingface": {
      "command": "npx",
      "args": ["@huggingface/mcp-server"],
      "env": {
        "HF_TOKEN": "hf_your_token_here"
      }
    }
  }
}
```

## Use Cases
1. Discover and evaluate pre-trained models for specific NLP/CV tasks before fine-tuning
2. Search datasets for training data and verify schema compatibility with pipelines
3. Run quick inference tests on candidate models to assess quality before deployment

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Port models to TAO | tao-port-huggingface-model | Transfer HuggingFace models into TAO Toolkit for optimization |
| Fine-tune HF models | tao-finetune-huggingface-model | Fine-tune discovered HuggingFace models using TAO training |

## Prerequisites
- `HF_TOKEN` (optional for public models, required for private/gated models)
- Node.js 18+ (for npx)

## Security Notes
- HF_TOKEN with write access can modify your repositories; use read-only tokens when possible
- Gated model access requires accepting license agreements on huggingface.co

## References
- https://github.com/mcp/huggingface/hf-mcp-server
- https://huggingface.co/docs/hub/
- https://huggingface.co/mcp
