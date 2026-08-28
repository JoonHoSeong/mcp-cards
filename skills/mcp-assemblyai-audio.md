# AssemblyAI Audio Intelligence MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AssemblyAI Audio Intelligence MCP Server |
| **Category** | AI/Speech |
| **Official** | ⭐ Official (AssemblyAI) |
| **Source** | https://github.com/AssemblyAI/assemblyai-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @assemblyai/mcp-server` |

## Tools & Capabilities
- `transcribe_audio` — High-accuracy speech-to-text with Conformer-2 architecture
- `identify_speakers` — Multi-speaker diarization with label assignment
- `detect_pii` — Detect and redact sensitive personal identifiable information (PII)
- `extract_key_phrases` — Extract salient key phrases and action items from conversations
- `lemur_query` — Run custom LLM reasoning (LeMUR) on audio transcripts

## Client Configuration
```json
{
  "mcpServers": {
    "assemblyai": {
      "command": "npx",
      "args": [
        "-y",
        "@assemblyai/mcp-server"
      ],
      "env": {
        "ASSEMBLYAI_API_KEY": "YOUR_ASSEMBLYAI_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enable PII redaction by default for enterprise compliance. Transmit audio over TLS.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
