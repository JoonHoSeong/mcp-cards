# CircleCI MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | CircleCI MCP |
| **Category** | CI/CD |
| **Official** | Yes (CircleCI) |
| **Source** | https://github.com/circleci-public/mcp-server-circleci |
| **Transport** | stdio |
| **Install** | `npx @circleci/mcp-server` |

## Description
The CircleCI MCP Server connects AI agents to CircleCI's continuous integration and delivery platform. It enables triggering pipelines, monitoring workflow status, retrieving test results and artifacts, and rerunning failed workflows — providing automated CI/CD orchestration for ML framework testing, model validation, and deployment pipelines.

## Key Tools
| Tool | Description |
|------|-------------|
| `get_pipeline_status` | Get the status of a pipeline execution |
| `list_workflows` | List workflows within a pipeline |
| `get_workflow_jobs` | Get jobs within a specific workflow |
| `trigger_pipeline` | Trigger a new pipeline run |
| `get_job_artifacts` | Download artifacts from a completed job |
| `list_projects` | List all followed projects |
| `get_test_results` | Retrieve test results for a job |
| `rerun_workflow` | Rerun a failed or cancelled workflow |

## Configuration
```json
{
  "mcpServers": {
    "circleci": {
      "command": "npx",
      "args": ["@circleci/mcp-server"],
      "env": {
        "CIRCLECI_TOKEN": "your-personal-api-token"
      }
    }
  }
}
```

## Use Cases
1. Trigger and monitor ML framework test suites on GPU runners
2. Retrieve model evaluation artifacts from training pipelines
3. Automatically rerun flaky tests in distributed training CI

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| ML framework testing | mcore-testing | Run Megatron-Core unit and integration tests in CI |
| Test monitoring | mcore-testing | Monitor GPU test job status and retrieve results |
| Pipeline triggers | mcore-testing | Trigger test pipelines on model code changes |

## Prerequisites
- CircleCI account with projects configured
- `CIRCLECI_TOKEN` personal API token
- GPU resource class enabled for ML workloads (optional)

## Security Notes
- Use project-scoped tokens when possible
- Store secrets in CircleCI contexts, not pipeline config
- Enable restricted contexts for production deployments
- Review pipeline triggers for unauthorized access

## References
- https://github.com/circleci-public/mcp-server-circleci
- https://circleci.com/docs/api/v2/
- https://circleci.com/docs/pipelines/
