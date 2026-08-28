# AWS Lambda Serverless MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AWS Lambda Serverless MCP Server |
| **Category** | Cloud/Serverless |
| **Official** | ⭐ Official (AWS Labs) |
| **Source** | https://github.com/awslabs/mcp-lambda |
| **Transport** | stdio |
| **Install** | `npx -y @awslabs/mcp-lambda` |

## Tools & Capabilities
- `list_functions` — Retrieve all Lambda functions, runtimes, memory, and timeouts
- `invoke_function` — Trigger synchronous or asynchronous function invocation with payload
- `get_function_logs` — Fetch CloudWatch execution log streams and duration/memory stats
- `update_function_code` — Deploy updated ZIP or container image function package
- `manage_event_mappings` — List and configure SQS, DynamoDB Streams, and Kinesis triggers

## Client Configuration
```json
{
  "mcpServers": {
    "aws-lambda": {
      "command": "npx",
      "args": [
        "-y",
        "@awslabs/mcp-lambda"
      ],
      "env": {
        "AWS_REGION": "us-east-1",
        "AWS_ACCESS_KEY_ID": "YOUR_AWS_ACCESS_KEY",
        "AWS_SECRET_ACCESS_KEY": "YOUR_AWS_SECRET_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Validate input payload schema before invocation. Require explicit approval for code updates.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
