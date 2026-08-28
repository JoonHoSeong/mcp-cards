# Google Maps Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Google Maps Platform MCP Server |
| **Category** | Location/Geospatial |
| **Official** | ⭐ Official (Google Maps) |
| **Source** | https://github.com/googlemaps/mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @googlemaps/mcp-server` |

## Tools & Capabilities
- `geocode_address` — Convert address text to latitude/longitude coordinates
- `reverse_geocode` — Convert coordinates to structured physical address
- `calculate_directions` — Compute optimal route, distance, and duration across transit modes
- `search_places` — Find nearby businesses, ratings, opening hours, and place details
- `get_elevation` — Query ground elevation data for specified coordinates

## Client Configuration
```json
{
  "mcpServers": {
    "google-maps": {
      "command": "npx",
      "args": [
        "-y",
        "@googlemaps/mcp-server"
      ],
      "env": {
        "GOOGLE_MAPS_API_KEY": "YOUR_GOOGLE_MAPS_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Restrict Google Maps API key by HTTP referrers or IP address. Monitor Places API usage quotas.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
