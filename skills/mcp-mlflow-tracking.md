# MLflow Tracking MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mlflow-mcp |
| **Category** | AI/ML |
| **Official** | Yes |
| **Source** | https://mlflow.org/docs/latest/genai/mcp/index.html |
| **Transport** | stdio |
| **Install** | `pip install mlflow[mcp]` |

## Description
MLflow MCP Server provides AI agents with direct access to MLflow's experiment tracking, model registry, and run management capabilities. It enables LLM-powered workflows to search experiments, compare model runs, log metrics, and manage model lifecycle stages — all through the Model Context Protocol without leaving the IDE.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_experiments` | List all MLflow experiments in the tracking server |
| `search_runs` | Search runs with filters across experiments |
| `get_run` | Get detailed information about a specific run |
| `compare_runs` | Compare metrics and parameters across multiple runs |
| `log_metric` | Log a metric value to an active run |
| `list_registered_models` | List all models in the model registry |
| `get_model_version` | Get details of a specific model version |
| `transition_model_stage` | Transition a model version between stages |

## Configuration
```json
{
  "mcpServers": {
    "mlflow": {
      "command": "mlflow",
      "args": ["mcp", "server"],
      "env": {
        "MLFLOW_TRACKING_URI": "http://localhost:5000"
      }
    }
  }
}
```

## Use Cases
1. Query and compare experiment results across distributed training runs from within an IDE
2. Automate model promotion workflows by transitioning model stages via natural language
3. Log metrics and track experiment progress during interactive development sessions

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Track distributed experiments | nemo-automodel-distributed-training | Monitor metrics across multi-GPU/multi-node NeMo training jobs |
| Track AutoML runs | tao-run-automl | Log and compare TAO AutoML hyperparameter search results |

## Prerequisites
- `MLFLOW_TRACKING_URI` environment variable pointing to your MLflow server
- MLflow server running (local or remote)
- Python 3.8+

## Security Notes
- Ensure MLflow tracking server is properly authenticated in production environments
- API tokens should be stored securely and never committed to version control

## References
- https://mlflow.org/docs/latest/genai/mcp/index.html
- https://mlflow.org/docs/latest/tracking.html
