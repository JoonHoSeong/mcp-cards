# OpenTelemetry MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | OpenTelemetry MCP Server |
| **Category** | Observability/Tracing |
| **Official** | ⭐ Official (Traceloop) |
| **Source** | https://github.com/traceloop/opentelemetry-mcp-server |
| **Transport** | stdio |
| **Install** | `pip install opentelemetry-mcp-server` |

## Description
MCP server from Traceloop that provides AI agents with access to OpenTelemetry trace data. Enables debugging distributed systems, analyzing LLM call traces, finding error patterns, and monitoring AI pipeline health through standard OpenTelemetry-compatible backends like Jaeger, Tempo, and Traceloop.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_traces` | Search traces by service, operation, tags, or duration |
| `get_trace` | Retrieve a complete trace by trace ID |
| `get_span` | Get detailed information about a specific span |
| `list_services` | List all services reporting traces |
| `get_service_metrics` | Get latency, throughput, and error rate for a service |
| `query_logs` | Query structured logs correlated with traces |
| `get_error_traces` | Find traces containing errors or exceptions |
| `compare_latency` | Compare latency between time periods or service versions |
| `get_llm_traces` | Retrieve traces specific to LLM/AI model calls |

## Configuration
```json
{
  "mcpServers": {
    "opentelemetry": {
      "command": "opentelemetry-mcp-server",
      "args": [],
      "env": {
        "OTEL_BACKEND_URL": "http://localhost:16686",
        "OTEL_BACKEND_TYPE": "jaeger",
        "TRACELOOP_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

## Use Cases
1. Debug distributed systems by tracing requests across services
2. Analyze LLM call traces including token usage and latency
3. Find error patterns and root causes in microservice architectures
4. Compare latency across services before and after deployments
5. Monitor AI pipeline health and identify bottlenecks

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Trace distributed training | nemo-automodel-distributed-training | Monitor communication patterns and bottlenecks in distributed training |
| Trace research agent calls | aiq-research | Analyze AI-Q research agent execution traces and LLM call chains |
| Trace RAG pipeline latency | rag-perf | Identify latency bottlenecks in RAG retrieval and generation stages |

## Prerequisites
- OpenTelemetry-compatible trace backend (Jaeger, Grafana Tempo, or Traceloop Cloud)
- Backend URL accessible from the development environment
- API key (for cloud-hosted backends like Traceloop)
- Python 3.9+ for pip installation

## Security Notes
- Use read-only access to the trace backend — trace data is observational
- API keys for cloud backends should be scoped to read operations
- Trace data may contain sensitive information (headers, parameters) — handle accordingly
- Ensure trace backend is not exposed to public networks
- Use separate backends or namespaces for development vs. production traces

## References
- https://github.com/traceloop/opentelemetry-mcp-server
- https://opentelemetry.io/docs/
- https://www.jaegertracing.io/docs/
- https://grafana.com/docs/tempo/latest/
- https://www.traceloop.com/docs
