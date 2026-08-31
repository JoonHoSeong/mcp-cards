# Hugging Face Datasets MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Hugging Face Datasets MCP Server |
| **Category** | AI/Data |
| **Official** | ⭐ Official (Hugging Face) |
| **Source** | https://github.com/huggingface/mcp-datasets |
| **Transport** | stdio |
| **Install** | `npx -y @huggingface/mcp-datasets` |

## Tools & Capabilities
- `preview_dataset_rows` — Stream first N rows of Parquet/Arrow datasets without full download
- `search_datasets` — Query datasets by modality, license, language, and task category
- `inspect_dataset_schema` — Retrieve column names, feature datatypes, and split sizes
- `filter_and_export` — Filter dataset subsets using SQL-like expressions

## Client Configuration
```json
{
  "mcpServers": {
    "hf-datasets": {
      "command": "npx",
      "args": [
        "-y",
        "@huggingface/mcp-datasets"
      ],
      "env": {
        "HF_TOKEN": "hf_..."
      }
    }
  }
}
```

## Security & Best Practices
- Read-only dataset streaming with memory-efficient chunking.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
