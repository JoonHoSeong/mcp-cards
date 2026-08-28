# Langflow Workflow MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | langflow-mcp |
| **Category** | AI/ML |
| **Official** | Community |
| **Source** | https://github.com/nobrainer-tech/langflow-mcp |
| **Transport** | stdio |
| **Install** | `npx langflow-mcp-server` |

## Description
Langflow MCP Server exposes Langflow's visual AI workflow builder to AI agents via the Model Context Protocol. With 93 available tools, it enables LLM workflows to create, execute, and monitor Langflow flows programmatically — turning the drag-and-drop workflow builder into an API-driven orchestration layer that agents can compose and trigger from within the IDE.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_flows` | List all available Langflow flows |
| `run_flow` | Execute a flow with specified inputs |
| `get_flow_status` | Check the execution status of a running flow |
| `list_components` | List available components for flow building |
| `create_flow` | Create a new flow from component definitions |
| `upload_file` | Upload a file for use in flow execution |
| `get_flow_output` | Retrieve the output of a completed flow |

## Configuration
```json
{
  "mcpServers": {
    "langflow": {
      "command": "npx",
      "args": ["langflow-mcp-server"],
      "env": {
        "LANGFLOW_URL": "http://localhost:7860",
        "LANGFLOW_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

## Use Cases
1. Orchestrate complex multi-step AI workflows combining RAG, agents, and tools
2. Programmatically create and iterate on flow designs during rapid prototyping
3. Trigger production Langflow pipelines from development environments for testing

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Orchestrate AI workflows | aiq-deploy | Deploy AIQ agents as components within Langflow orchestration flows |
| Design training data | nemo-data-designer-plugin | Integrate NeMo data generation into Langflow data pipelines |

## Prerequisites
- `LANGFLOW_URL` pointing to your Langflow instance
- `LANGFLOW_API_KEY` for authenticated access
- Langflow server running (local or cloud)
- Node.js 18+ (for npx)

## Security Notes
- Flows may execute arbitrary code; ensure Langflow instance is properly sandboxed
- Community-maintained server — review source before deploying in production

## References
- https://github.com/nobrainer-tech/langflow-mcp
- https://docs.langflow.org/
