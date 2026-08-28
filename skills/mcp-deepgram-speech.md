# Deepgram Speech Recognition MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Deepgram Speech Recognition MCP Server |
| **Category** | AI/Speech |
| **Official** | ⭐ Official (Deepgram) |
| **Source** | https://github.com/deepgram/mcp-deepgram |
| **Transport** | stdio |
| **Install** | `npx -y @deepgram/mcp-server` |

## Tools & Capabilities
- `transcribe_audio_url` — Transcribe remote audio/video URL using Nova-2 / Nova-3 models
- `transcribe_file` — Stream and transcribe local audio file with speaker diarization
- `generate_summary` — Produce meeting and conversation summaries from audio transcripts
- `detect_topics` — Identify key topics and sentiment polarity across spoken dialogue
- `speak_text` — Convert text to low-latency streaming speech using Deepgram Aura

## Client Configuration
```json
{
  "mcpServers": {
    "deepgram": {
      "command": "npx",
      "args": [
        "-y",
        "@deepgram/mcp-server"
      ],
      "env": {
        "DEEPGRAM_API_KEY": "YOUR_DEEPGRAM_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce HTTPS endpoints for audio file URLs. Sanitize transcribed PII where required.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
