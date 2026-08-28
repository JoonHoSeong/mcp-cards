# SonarQube MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | SonarQube MCP Server |
| **Category** | Security/Code Quality |
| **Official** | ⭐ Official (SonarSource) |
| **Source** | https://github.com/SonarSource/sonarqube-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @sonarqube/mcp-server` |

## Description
Official MCP server from SonarSource that brings SonarQube code quality analysis capabilities to AI agents. Enables scanning code for bugs, vulnerabilities, and code smells, checking quality gates, and reviewing security hotspots directly from the development environment.

## Key Tools
| Tool | Description |
|------|-------------|
| `analyze_code` | Run SonarQube analysis on code snippets or files |
| `get_issues` | Retrieve issues (bugs, vulnerabilities, code smells) for a project |
| `get_quality_gate_status` | Check if a project passes its quality gate |
| `list_projects` | List all projects configured in SonarQube |
| `get_metrics` | Get code metrics (coverage, duplication, complexity) |
| `get_hotspots` | Retrieve security hotspots requiring review |
| `search_rules` | Search available analysis rules by language or type |
| `get_coverage` | Get code coverage details for a project |

## Configuration
```json
{
  "mcpServers": {
    "sonarqube": {
      "command": "npx",
      "args": ["-y", "@sonarqube/mcp-server"],
      "env": {
        "SONARQUBE_URL": "https://sonarqube.example.com",
        "SONARQUBE_TOKEN": "squ_your-token-here"
      }
    }
  }
}
```

## Use Cases
1. Scan code for bugs and vulnerabilities before committing
2. Check quality gates before merging pull requests
3. Analyze code snippets in context for immediate feedback
4. Track and reduce technical debt across projects
5. Review security hotspots and prioritize remediation

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Ensure ML framework quality | mcore-linting-and-formatting | Complement Megatron-Core linting with deeper static analysis |
| Pipeline code quality | deepstream-dev | Validate DeepStream pipeline code for security and reliability |
| Validate generated code | skill-card-generator | Quality-check auto-generated skill card code |

## Prerequisites
- SonarQube Server (v9.9+) or SonarQube Cloud account
- Server URL accessible from the development environment
- User token with `Browse` permission on target projects
- Node.js 18+ for npx execution

## Security Notes
- Use project-scoped tokens with read-only permissions for analysis
- Avoid using admin tokens — create dedicated service accounts
- Token provides access to all projects the user can browse; scope appropriately
- SonarQube Cloud tokens are scoped to organization level
- Do not commit tokens to version control

## References
- https://github.com/SonarSource/sonarqube-mcp-server
- https://docs.sonarsource.com/sonarqube/latest/
- https://docs.sonarsource.com/sonarqube/latest/user-guide/user-account/generating-and-using-tokens/
