# Figma MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Figma Context MCP |
| **Category** | Design |
| **Official** | Community |
| **Source** | https://github.com/GLips/Figma-Context-MCP |
| **Transport** | stdio |
| **Install** | `npx -y figma-context-mcp` |

## Description
Figma Context MCP server provides AI agents with access to Figma design files, components, styles, and design tokens. It bridges design and development workflows by enabling automated extraction of design specifications, asset export, and component inspection directly from Figma files.

## Key Tools
| Tool | Description |
|------|-------------|
| `get_file` | Retrieve full Figma file structure and metadata |
| `get_components` | List all components and their properties in a file |
| `get_styles` | Extract color, text, and effect styles from a file |
| `get_frame` | Get specific frame data including layout and children |
| `export_assets` | Export design assets in specified formats (PNG, SVG, PDF) |
| `get_design_tokens` | Extract design tokens for colors, spacing, and typography |

## Configuration
```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "figma-context-mcp"],
      "env": {
        "FIGMA_PERSONAL_ACCESS_TOKEN": "<your-token>"
      }
    }
  }
}
```

## Use Cases
1. Generate code from design specifications automatically  2. Extract design tokens for theming systems  3. Audit component usage and design consistency

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Design-to-3D pipeline | omniverse-realtime-viewer | Convert Figma UI designs into Omniverse 3D scene layouts |
| Dashboard UI generation | vss-setup-video-analytics-api | Generate video analytics dashboard code from Figma mockups |

## Prerequisites
- `FIGMA_PERSONAL_ACCESS_TOKEN` with file read access
- Figma file URL or key for target designs

## Security Notes
- Token provides read access to all team files; use file-scoped tokens when possible
- Exported assets may contain proprietary design IP; secure output directories
- Rate limits apply; cache responses for repeated queries

## References
- https://github.com/GLips/Figma-Context-MCP
- https://www.figma.com/developers/api
