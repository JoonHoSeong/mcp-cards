# Chrome DevTools Protocol MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Chrome DevTools Protocol MCP Server |
| **Category** | Developer/Debugging |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/chrome-devtools |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-chrome-devtools` |

## Tools & Capabilities
- `inspect_network_traffic` — Intercept and analyze HTTP requests, responses, headers, and payload timings
- `evaluate_js_expression` — Execute JavaScript in the active page execution context and return evaluated values
- `get_dom_tree` — Retrieve sanitized DOM tree hierarchy and computed CSS styles
- `capture_performance_trace` — Record timeline profiles for Core Web Vitals and JavaScript execution bottlenecks
- `emulate_device_metrics` — Emulate mobile screen resolutions, user-agent strings, and network throttling

## Client Configuration
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-chrome-devtools",
        "--remote-debugging-port=9222"
      ]
    }
  }
}
```

## Security & Best Practices
- Local Chrome remote debugging port (9222). Do not expose remote debugging ports to public networks.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
