# Ethereum / EVM Blockchain MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Ethereum / EVM Blockchain MCP Server |
| **Category** | Web3/Blockchain |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/evm/mcp-ethereum |
| **Transport** | stdio |
| **Install** | `npx -y @evm/mcp-server` |

## Tools & Capabilities
- `get_eth_balance` — Query native ETH and ERC-20 token balances for wallet address
- `call_contract_view` — Read public view/pure functions on verified smart contracts using ABI
- `get_transaction_receipt` — Fetch gas used, event logs, and status for transaction hash
- `simulate_call` — Simulate contract execution using `eth_call` with state overrides
- `get_gas_price` — Fetch live EIP-1559 base fee and priority fee suggestions

## Client Configuration
```json
{
  "mcpServers": {
    "ethereum": {
      "command": "npx",
      "args": [
        "-y",
        "@evm/mcp-server"
      ],
      "env": {
        "ETH_RPC_URL": "https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Strict read-only JSON-RPC calls. Never expose wallet seed phrases or private keys.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
