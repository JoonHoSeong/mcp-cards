# Weights & Biases MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Weights & Biases MCP Server |
| **Category** | AI/ML Platform |
| **Official** | ⭐ Official (W&B) |
| **Source** | https://github.com/wandb/wandb-mcp-server |
| **Transport** | stdio |
| **Install** | `pip install wandb-mcp-server` |

## Description
MCP server for Weights & Biases experiment tracking platform. Enables AI agents to query ML experiments, compare training runs, analyze hyperparameter sweeps, and track artifact lineage directly from the IDE. Provides read-only access to W&B project data including runs, metrics, sweeps, and artifacts.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_runs` | Search and filter experiment runs by name, tags, or metrics |
| `get_run_metrics` | Retrieve detailed metrics and logs for a specific run |
| `compare_runs` | Compare metrics across multiple training runs |
| `list_artifacts` | List all artifacts associated with a project or run |
| `get_artifact` | Get metadata and details of a specific artifact |
| `list_sweeps` | List hyperparameter sweep configurations |
| `get_sweep_results` | Get results and best configurations from a sweep |
| `query_traces` | Query W&B Traces for LLM observability |
| `list_projects` | List all projects in a W&B entity/team |

## Configuration
```json
{
  "mcpServers": {
    "wandb": {
      "command": "wandb-mcp-server",
      "env": {
        "WANDB_API_KEY": "your-wandb-api-key",
        "WANDB_BASE_URL": "https://api.wandb.ai"
      }
    }
  }
}
```

## Use Cases
1. Query ML experiments from IDE without switching to the W&B dashboard
2. Compare model training runs to identify best hyperparameters
3. Analyze hyperparameter sweeps and find optimal configurations
4. Debug training failures by examining run logs and system metrics
5. Track artifact lineage across model versions and datasets

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Track distributed training metrics | `nemo-automodel-distributed-training` | Monitor multi-GPU/multi-node NeMo training runs and compare convergence |
| Track AutoML experiments | `tao-run-automl` | Log TAO AutoML trials and analyze architecture search results |
| Track RL experiments | `nemo-rl-auto-research` | Monitor reward curves and policy training across RL experiments |

## Prerequisites
- `WANDB_API_KEY` environment variable set
- Active Weights & Biases account (free tier available)
- Python 3.8+ for installation
- Existing W&B projects with logged runs

## Security Notes
- API key can be scoped per project for minimal access
- Read-only access by default — no write operations to experiments
- Store API key in environment variables, never in config files
- Supports team-level access controls from W&B organization settings

## References
- [W&B MCP Server Documentation](https://docs.wandb.ai/platform/mcp-server)
- [GitHub Repository](https://github.com/wandb/wandb-mcp-server)
- [W&B API Documentation](https://docs.wandb.ai/ref/python)
