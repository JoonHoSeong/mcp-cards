# Jenkins MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Jenkins MCP |
| **Category** | CI/CD |
| **Official** | No (Community) |
| **Source** | https://github.com/hekmon8/Jenkins-server-mcp |
| **Transport** | stdio |
| **Install** | `npx jenkins-mcp-server` |

## Description
The Jenkins MCP Server enables AI agents to interact with Jenkins CI/CD servers for build automation. It provides tools for triggering builds, monitoring job status, retrieving build logs, and managing the build queue — allowing agents to orchestrate training workflows, run validation pipelines, and manage enterprise ML build infrastructure.

## Key Tools
| Tool | Description |
|------|-------------|
| `get_build_status` | Get the status of a specific build |
| `trigger_build` | Trigger a new build for a job |
| `get_build_log` | Retrieve console output from a build |
| `list_jobs` | List all Jenkins jobs |
| `get_job_config` | Get the XML configuration of a job |
| `list_builds` | List recent builds for a job |
| `abort_build` | Abort a running build |
| `get_queue` | Get the current build queue |

## Configuration
```json
{
  "mcpServers": {
    "jenkins": {
      "command": "npx",
      "args": ["jenkins-mcp-server"],
      "env": {
        "JENKINS_URL": "https://jenkins.your-domain.com",
        "JENKINS_USER": "your-username",
        "JENKINS_TOKEN": "your-api-token"
      }
    }
  }
}
```

## Use Cases
1. Trigger model training jobs on GPU-equipped Jenkins agents
2. Monitor training pipeline progress and retrieve metrics
3. Abort long-running builds that exceed cost or time budgets

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Training triggers | tao-launch-workflow | Trigger TAO training pipelines via Jenkins jobs |
| Build monitoring | tao-launch-workflow | Monitor training job status and completion |
| Log retrieval | tao-launch-workflow | Retrieve training logs for debugging failures |

## Prerequisites
- Jenkins server accessible over HTTP/HTTPS
- `JENKINS_URL` base URL of the Jenkins instance
- `JENKINS_USER` and `JENKINS_TOKEN` API credentials

## Security Notes
- Use API tokens instead of passwords for authentication
- Restrict token permissions to required jobs only
- Enable CSRF protection on the Jenkins server
- Use HTTPS for all Jenkins API communication

## References
- https://github.com/hekmon8/Jenkins-server-mcp
- https://www.jenkins.io/doc/book/using/remote-access-api/
- https://www.jenkins.io/doc/
