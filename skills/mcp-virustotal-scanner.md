# VirusTotal Threat Intelligence MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | VirusTotal Threat Intelligence MCP Server |
| **Category** | Security/Threat-Intel |
| **Official** | ⭐ Official (VirusTotal / GTI) |
| **Source** | https://github.com/virustotal/mcp-virustotal |
| **Transport** | stdio |
| **Install** | `npx -y @virustotal/mcp-server` |

## Tools & Capabilities
- `scan_url` — Submit URL for analysis by 70+ antivirus engines and phishing databases
- `get_file_report` — Inspect SHA-256 hash threat score, detection names, and behavior sandboxes
- `inspect_ip_address` — Query IP address reputation, autonomous system (ASN), and passive DNS records
- `get_domain_report` — Check domain WHOIS, malware associations, and SSL certificate validity

## Client Configuration
```json
{
  "mcpServers": {
    "virustotal": {
      "command": "npx",
      "args": [
        "-y",
        "@virustotal/mcp-server"
      ],
      "env": {
        "VIRUSTOTAL_API_KEY": "YOUR_VT_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce VirusTotal API rate limits (4 requests/min for standard keys).

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
