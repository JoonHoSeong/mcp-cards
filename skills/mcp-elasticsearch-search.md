# Elasticsearch MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-elasticsearch-search |
| **Category** | Database |
| **Official** | Official (Elastic) |
| **Source** | https://github.com/elastic/mcp-server-elasticsearch |
| **Transport** | stdio |
| **Install** | `docker pull docker.elastic.co/elasticsearch/mcp-server-elasticsearch` |

## Description
The Elasticsearch MCP server provides AI agents with full-text search, analytics, and document indexing capabilities through Elasticsearch clusters. Available as a Docker container, it supports complex queries, aggregations, bulk operations, and mapping management. It is particularly effective for log analysis, content search, and building search-powered AI applications at scale.

## Key Tools
| Tool | Description |
|------|-------------|
| `search` | Execute search queries with full Query DSL support |
| `index_document` | Index a single document into a specified index |
| `get_document` | Retrieve a document by its ID |
| `create_index` | Create a new index with mappings and settings |
| `get_mapping` | Get field mappings for an index |
| `aggregate` | Run aggregation queries for analytics |
| `bulk_index` | Index multiple documents in a single bulk operation |

## Configuration
```json
{
  "mcpServers": {
    "elasticsearch": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "docker.elastic.co/elasticsearch/mcp-server-elasticsearch"],
      "env": {
        "ES_URL": "https://your-cluster.es.cloud:9243",
        "ES_API_KEY": "your-elasticsearch-api-key"
      }
    }
  }
}
```

## Use Cases
1. Full-text search across large document collections and knowledge bases
2. Log aggregation and analytics for observability pipelines
3. Indexing and searching video metadata for media archives

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Video search index | vss-search-archive | Index and search video scene metadata and transcripts for Visual Search & Summarization |

## Prerequisites
- Elasticsearch cluster (self-hosted, Elastic Cloud, or AWS OpenSearch)
- `ES_URL` cluster endpoint
- `ES_API_KEY` with appropriate index permissions
- Docker runtime for container-based installation

## Security Notes
- Use API keys with minimal required privileges scoped to specific indices
- Enable TLS for cluster communication and disable anonymous access
- Avoid exposing cluster endpoints to public networks without authentication

## References
- https://github.com/elastic/mcp-server-elasticsearch
- https://www.elastic.co/docs
