# Apache Kafka Streaming MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Apache Kafka Streaming MCP Server |
| **Category** | Data/Streaming |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/kafka |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-kafka` |

## Tools & Capabilities
- `list_topics` — Query active Kafka topics, partition counts, and replication factors
- `produce_message` — Publish structured JSON/Avro message to specified Kafka topic partition
- `consume_messages` — Stream and inspect latest N messages from designated topic
- `inspect_consumer_groups` — Monitor consumer lag, group rebalancing states, and committed offsets

## Client Configuration
```json
{
  "mcpServers": {
    "kafka": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-kafka"
      ],
      "env": {
        "KAFKA_BROKERS": "localhost:9092",
        "KAFKA_CLIENT_ID": "mcp-agent"
      }
    }
  }
}
```

## Security & Best Practices
- SASL/SCRAM or mTLS authentication. Enforce consumer message limit bounds.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
