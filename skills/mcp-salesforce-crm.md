# Salesforce CRM & Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Salesforce CRM & Platform MCP Server |
| **Category** | Enterprise/CRM |
| **Official** | ⭐ Official (Salesforce) |
| **Source** | https://github.com/salesforce/mcp-salesforce |
| **Transport** | stdio |
| **Install** | `npx -y @salesforce/mcp-server` |

## Tools & Capabilities
- `execute_soql` — Query Salesforce objects (Accounts, Contacts, Leads, Opportunities) using SOQL
- `create_record` — Insert new CRM record with custom field mappings and validation rule compliance
- `update_record` — Update existing object fields by record ID
- `describe_sobject` — Inspect field types, picklist values, and relational schemas for objects
- `invoke_apex_action` — Trigger server-side Apex REST endpoints and Flow automations

## Client Configuration
```json
{
  "mcpServers": {
    "salesforce": {
      "command": "npx",
      "args": [
        "-y",
        "@salesforce/mcp-server"
      ],
      "env": {
        "SALESFORCE_INSTANCE_URL": "https://your-org.my.salesforce.com",
        "SALESFORCE_ACCESS_TOKEN": "YOUR_ACCESS_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Strict Field-Level Security (FLS) enforcement. SOQL queries must be parameterized.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
