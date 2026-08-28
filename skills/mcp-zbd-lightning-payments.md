# ZBD MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | zbd-payments-mcp |
| **Category** | Payments/Billing |
| **Official** | Yes (ZBD/Zebedee) |
| **Source** | https://github.com/zebedeeio/zbd-payments-typescript-sdk |
| **Transport** | stdio |
| **Install** | `npm install @zbddev/mcp-server` |

## Description
ZBD's official MCP server enables AI agents to send and receive Bitcoin Lightning Network payments instantly and globally. With sub-second settlement, near-zero fees, and micropayment support (fractions of a cent), it's ideal for gaming rewards, tipping, pay-per-use AI services, and cross-border instant transfers without traditional banking rails.

## Key Tools
| Tool | Description |
|------|-------------|
| `send_payment` | Send instant Bitcoin Lightning payment |
| `create_charge` | Create a payment request (invoice) |
| `get_wallet_balance` | Check current wallet balance |
| `list_transactions` | List recent transactions |
| `create_withdrawal` | Create a withdrawal request |
| `send_to_gamertag` | Send payment to a ZBD gamertag |
| `send_to_email` | Send payment to an email address |

## Configuration
```json
{
  "mcpServers": {
    "zbd": {
      "command": "npx",
      "args": ["@zbddev/mcp-server"],
      "env": {
        "ZBD_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Use Cases
1. AI agent distributing micropayments for gaming rewards and achievements
2. Pay-per-use inference billing with instant Lightning settlement
3. Cross-border instant payments without traditional banking intermediaries

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Crypto portfolio | portfolio-optimization | Integrate Lightning payments into crypto strategies |
| Gaming AI | nemotron-reasoning | Intelligent reward distribution in gaming contexts |
| Micropayment billing | aiq-deploy | Charge per-inference via Lightning micropayments |

## Prerequisites
- ZBD developer account
- `ZBD_API_KEY` — API key from ZBD Developer Dashboard
- Node.js 18+

## Security Notes
- Set spending limits on API keys
- Monitor wallet balance and set low-balance alerts
- Lightning payments are irreversible — validate amounts before sending
- Use IP allowlisting for production API keys

## References
- https://github.com/zebedeeio/zbd-payments-typescript-sdk
- https://docs.zebedee.io/
- https://zbd.gg/developers
