# Confluent Kafka MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-confluent |
| **Category** | Streaming |
| **Official** | Yes |
| **Source** | https://github.com/confluentinc/mcp-confluent |
| **Transport** | stdio |
| **Install** | `npx @confluentinc/mcp-confluent` |

## Description
Official Confluent MCP server for interacting with Apache Kafka clusters on Confluent Cloud. Provides AI-assisted topic management, message production and consumption, schema registry access, and consumer group monitoring for event-driven architectures.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_topics` | List all topics in a Kafka cluster |
| `produce_message` | Produce a message to a topic |
| `consume_messages` | Consume messages from a topic |
| `list_clusters` | List available Kafka clusters |
| `get_topic_config` | Get configuration for a specific topic |
| `list_consumer_groups` | List consumer groups and their status |
| `get_schema` | Retrieve schema from Schema Registry |

## Configuration
```json
{
  "mcpServers": {
    "confluent": {
      "command": "npx",
      "args": ["@confluentinc/mcp-confluent"],
      "env": {
        "CONFLUENT_CLOUD_API_KEY": "your-api-key",
        "CONFLUENT_CLOUD_API_SECRET": "your-api-secret"
      }
    }
  }
}
```

## Use Cases
1. Stream video analytics events from VSS pipelines to downstream consumers
2. Monitor consumer group lag for inference result processing topics
3. Produce and consume test messages for validating event-driven AI pipelines

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Video event streaming | vss-manage-video-io-storage | Stream video analytics events through Kafka for storage and processing |

## Prerequisites
- Confluent Cloud account or self-managed Confluent Platform
- `CONFLUENT_CLOUD_API_KEY` — Cloud API key for cluster access
- `CONFLUENT_CLOUD_API_SECRET` — Corresponding API secret

## Security Notes
- Use service accounts with topic-level ACLs rather than organization admin keys
- Enable encryption in transit (TLS) for all Kafka connections

## References
- https://github.com/confluentinc/mcp-confluent
- https://docs.confluent.io/cloud/current/api.html
