# Cypress Testing MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Cypress Testing MCP Server |
| **Category** | Frontend/Testing |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/cypress-io/cypress-mcp |
| **Transport** | stdio |
| **Install** | `npx -y cypress-mcp-server` |

## Tools & Capabilities
- `run_spec` — Execute specific Cypress test spec files in headless mode
- `get_test_results` — Parse mocha/junit XML and JSON test execution summaries
- `capture_failure_artifacts` — Retrieve screenshots and video logs of failed assertions
- `generate_test_scaffold` — Create test skeletons from user story descriptions
- `inspect_dom_snapshot` — Examine DOM state at the exact time of test failure

## Client Configuration
```json
{
  "mcpServers": {
    "cypress": {
      "command": "npx",
      "args": [
        "-y",
        "cypress-mcp-server"
      ],
      "env": {
        "CYPRESS_RECORD_KEY": "YOUR_CYPRESS_RECORD_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Sandbox test runner execution. Do not execute arbitrary untrusted scripts in test environment.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
