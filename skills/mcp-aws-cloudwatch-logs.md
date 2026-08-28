# AWS CloudWatch Logs MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AWS CloudWatch Logs MCP Server |
| **Category** | Observability/Logs |
| **Official** | ⭐ Official (AWS Labs) |
| **Source** | https://github.com/awslabs/mcp-cloudwatch |
| **Transport** | stdio |
| **Install** | `npx -y @awslabs/mcp-cloudwatch` |

## Tools & Capabilities
- `describe_log_groups` — List log groups, retention policies, and stored byte metrics
- `filter_log_events` — Filter log streams by pattern matching, timestamp range, and limits
- `run_cloudwatch_insights_query` — Execute CloudWatch Logs Insights SQL-like queries
- `get_metric_data` — Retrieve CloudWatch metrics across EC2, Lambda, DynamoDB, and RDS
- `describe_alarms` — Inspect active alarm states, thresholds, and notification actions

## Client Configuration
```json
{
  "mcpServers": {
    "aws-cloudwatch": {
      "command": "npx",
      "args": [
        "-y",
        "@awslabs/mcp-cloudwatch"
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
- Enforce bounded query time ranges. Mask sensitive PII and authorization headers in log outputs.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
