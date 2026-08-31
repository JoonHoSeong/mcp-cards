# Jupyter Notebook & Kernel MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Jupyter Notebook & Kernel MCP Server |
| **Category** | Data/Notebook |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/jupyter/mcp-jupyter |
| **Transport** | stdio |
| **Install** | `npx -y @jupyter/mcp-server` |

## Tools & Capabilities
- `execute_cell` — Execute Python/R/Julia code cell in live Jupyter kernel and capture output & plots
- `read_notebook` — Parse `.ipynb` notebook structure, markdown text, code cells, and outputs
- `insert_cell` — Add new code or markdown cell at designated index position in notebook
- `restart_kernel` — Restart execution kernel and clear active variable states
- `list_variables` — Inspect active kernel memory variables, dataframes, and shapes

## Client Configuration
```json
{
  "mcpServers": {
    "jupyter": {
      "command": "npx",
      "args": [
        "-y",
        "@jupyter/mcp-server",
        "--server-url=http://localhost:8888"
      ]
    }
  }
}
```

## Security & Best Practices
- Local Jupyter server token authentication. Restrict kernel execution to local environment.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
