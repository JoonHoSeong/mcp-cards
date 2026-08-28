# AWS Cloud Unified MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AWS Cloud Unified MCP Server |
| **Category** | Cloud/Infrastructure |
| **Official** | ⭐ Official (AWS Labs) |
| **Source** | https://github.com/awslabs/mcp-aws |
| **Transport** | stdio |
| **Install** | `npx -y @awslabs/mcp-aws` |

## Tools & Capabilities
- `list_ec2_instances` — Query EC2 instances, states, IPs, and tags
- `get_ecs_services` — Inspect ECS clusters, task definitions, and running services
- `manage_iam_roles` — Read IAM policies, roles, and credential boundary attachments
- `inspect_vpc_topology` — Retrieve subnets, route tables, and security group rules
- `get_cost_and_usage` — Query AWS Cost Explorer for billing breakdown and anomalies

## Client Configuration
```json
{
  "mcpServers": {
    "aws-cloud": {
      "command": "npx",
      "args": [
        "-y",
        "@awslabs/mcp-aws"
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
- Enforce IAM least privilege. Mutation tools require explicit confirmation before resource termination.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
