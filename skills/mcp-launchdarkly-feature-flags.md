# LaunchDarkly MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | LaunchDarkly MCP |
| **Category** | Feature Flags |
| **Official** | Official |
| **Source** | https://github.com/launchdarkly/mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @launchdarkly/mcp-server` |

## Description
LaunchDarkly MCP server enables AI agents to manage feature flags, targeting rules, and experiments. It supports gradual rollouts, A/B testing, and environment-specific flag management, making it ideal for controlled deployment of AI models and features with real-time kill switches and percentage-based rollouts.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_flags` | List all feature flags in a project with status |
| `get_flag` | Get detailed flag configuration and targeting rules |
| `toggle_flag` | Enable or disable a feature flag in an environment |
| `create_flag` | Create a new feature flag with variations |
| `update_targeting` | Update targeting rules for specific user segments |
| `list_environments` | List available environments (dev, staging, prod) |
| `get_flag_status` | Get flag evaluation statistics and usage |
| `create_experiment` | Create an A/B experiment on a flag variation |

## Configuration
```json
{
  "mcpServers": {
    "launchdarkly": {
      "command": "npx",
      "args": ["-y", "@launchdarkly/mcp-server"],
      "env": {
        "LD_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

## Use Cases
1. Gradual rollout of new AI model versions to production users  2. A/B testing different model configurations with real traffic  3. Emergency kill switches for misbehaving model deployments

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Gradual model rollout | nemotron-customize | Control percentage-based rollout of customized Nemotron models |
| NIM version switching | nim-agent-blueprint | Feature flag NIM inference endpoints for canary deployments |
| Analytics feature gates | vss-setup-video-analytics-api | Gate new video analytics features behind flags |

## Prerequisites
- `LD_API_KEY` with appropriate access level (reader, writer, or admin)
- LaunchDarkly project and environments configured

## Security Notes
- API key with writer access can modify production flags; use reader keys for monitoring
- Flag changes take effect immediately; implement approval workflows for production
- Audit log tracks all flag changes for compliance

## References
- https://github.com/launchdarkly/mcp-server
- https://docs.launchdarkly.com/home/getting-started
