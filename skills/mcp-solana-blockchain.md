# Solana Blockchain MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Solana Blockchain MCP Server |
| **Category** | Web3/Blockchain |
| **Official** | ⭐ Official (Solana Foundation / Community) |
| **Source** | https://github.com/solana-developers/mcp-solana |
| **Transport** | stdio |
| **Install** | `npx -y @solana/mcp-server` |

## Tools & Capabilities
- `get_account_balance` — Query SOL and SPL token balances for public wallet address
- `get_transaction` — Fetch parsed transaction details, compute units, and instruction logs
- `inspect_program` — Disassemble and inspect Anchor/Rust on-chain program accounts and IDLs
- `simulate_transaction` — Simulate transaction execution to test logs and compute budget prior to broadcast
- `get_latest_blockhash` — Retrieve current blockhash and slot commitment status

## Client Configuration
```json
{
  "mcpServers": {
    "solana": {
      "command": "npx",
      "args": [
        "-y",
        "@solana/mcp-server"
      ],
      "env": {
        "SOLANA_RPC_URL": "https://api.mainnet-beta.solana.com"
      }
    }
  }
}
```

## Security & Best Practices
- Read-only RPC queries by default. Private keys must never be stored in plaintext or logged.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
