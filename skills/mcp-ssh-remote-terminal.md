# SSH Remote Terminal MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | SSH Remote Terminal MCP Server |
| **Category** | DevOps/SSH |
| **Official** | ⭐ Verified (developerz-ai/mcp-ssh) |
| **Source** | https://github.com/developerz-ai/mcp-ssh |
| **Transport** | stdio |
| **Install** | `npx -y @developerz-ai/mcp-ssh` |

## Tools & Capabilities
- `execute_command` — Execute remote shell commands via SSH with captured stdout, stderr, and exit codes
- `upload_file_sftp` — Transfer local files or configurations to remote server via SFTP
- `download_file_sftp` — Fetch remote server log files or generated data over SFTP
- `check_server_resources` — Query remote server CPU, RAM, disk, and top processes

## Client Configuration
```json
{
  "mcpServers": {
    "ssh": {
      "command": "npx",
      "args": [
        "-y",
        "@developerz-ai/mcp-ssh"
      ],
      "env": {
        "SSH_HOST": "server.example.com",
        "SSH_USER": "ubuntu",
        "SSH_KEY_PATH": "~/.ssh/id_rsa"
      }
    }
  }
}
```

## Security & Best Practices
- SSH key-based authentication with strict host key verification. Reject interactive sudo prompts.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
