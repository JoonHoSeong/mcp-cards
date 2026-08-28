# ElevenLabs Voice & Audio MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | ElevenLabs Voice & Audio MCP Server |
| **Category** | AI/Voice |
| **Official** | ⭐ Official (ElevenLabs) |
| **Source** | https://github.com/elevenlabs/elevenlabs-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @elevenlabs/mcp-server` |

## Tools & Capabilities
- `generate_speech` — Convert text to lifelike speech using Multilingual v2 / Flash v2.5
- `list_voices` — Query premade and custom cloned voice profiles with stability settings
- `clone_voice` — Create instant or professional voice clones from audio samples
- `sound_effects` — Generate realistic sound effects and audio ambiences from prompts
- `isolate_audio` — Remove background noise and isolate clean vocal tracks

## Client Configuration
```json
{
  "mcpServers": {
    "elevenlabs": {
      "command": "npx",
      "args": [
        "-y",
        "@elevenlabs/mcp-server"
      ],
      "env": {
        "ELEVENLABS_API_KEY": "YOUR_ELEVENLABS_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Mask API tokens in request payloads. Require user consent before initiating voice cloning operations.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
