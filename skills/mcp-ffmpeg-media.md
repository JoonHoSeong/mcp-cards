# FFmpeg Multimedia Processing MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | FFmpeg Multimedia Processing MCP Server |
| **Category** | Media/Video |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/video-creator/ffmpeg-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @video-creator/mcp-ffmpeg` |

## Tools & Capabilities
- `transcode_video` — Convert video codecs (H.264, H.265, AV1) and containers (MP4, MKV, WebM)
- `trim_media` — Extract clip from start to end timestamp without quality loss
- `extract_audio` — Strip audio streams to MP3, WAV, or AAC format
- `generate_thumbnail_sprites` — Produce thumbnail grids and preview GIFs from video timestamps
- `inspect_media_streams` — Parse ffprobe metadata, resolutions, bitrates, and audio channels

## Client Configuration
```json
{
  "mcpServers": {
    "ffmpeg": {
      "command": "npx",
      "args": [
        "-y",
        "@video-creator/mcp-ffmpeg"
      ]
    }
  }
}
```

## Security & Best Practices
- Local FFmpeg binary execution. Restrict file access to authorized media working directories.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
