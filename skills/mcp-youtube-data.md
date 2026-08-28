# YouTube Data MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | YouTube Data MCP Server |
| **Category** | Media/Content |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/youtube/mcp-youtube-data |
| **Transport** | stdio |
| **Install** | `npx -y @youtube/mcp-server` |

## Tools & Capabilities
- `search_videos` — Search YouTube videos by keyword, channel, published date, and order
- `get_video_transcript` — Fetch timed transcript captions (subtitles) for video analysis
- `get_video_metadata` — Inspect view counts, likes, tags, descriptions, and thumbnails
- `list_channel_playlists` — Retrieve public playlists and video sequences for channels
- `get_comment_threads` — Extract top-level comments and community sentiment

## Client Configuration
```json
{
  "mcpServers": {
    "youtube": {
      "command": "npx",
      "args": [
        "-y",
        "@youtube/mcp-server"
      ],
      "env": {
        "YOUTUBE_API_KEY": "YOUR_YOUTUBE_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Read-only access to public video and transcript metadata. Respect YouTube Data API quota units.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
