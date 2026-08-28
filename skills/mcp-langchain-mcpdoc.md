# LangChain mcpdoc MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcpdoc |
| **Category** | AI/ML |
| **Official** | Yes |
| **Source** | https://github.com/langchain-ai/mcpdoc |
| **Transport** | stdio |
| **Install** | `uvx mcpdoc` |

## Description
LangChain mcpdoc MCP Server exposes library documentation via the llms.txt standard to AI agents and IDEs. It allows LLM workflows to resolve library names, search documentation, and retrieve relevant docs — ensuring agents always reference the latest SDK documentation rather than relying on potentially outdated training data. Supports any library that publishes an llms.txt file.

## Key Tools
| Tool | Description |
|------|-------------|
| `resolve_library` | Resolve a library name to its llms.txt endpoint |
| `get_docs` | Retrieve documentation for a specific library topic |
| `search_docs` | Search across indexed library documentation |
| `get_llms_txt` | Fetch the raw llms.txt file for a library |

## Configuration
```json
{
  "mcpServers": {
    "mcpdoc": {
      "command": "uvx",
      "args": ["mcpdoc"]
    }
  }
}
```

## Use Cases
1. Retrieve up-to-date SDK documentation while coding against rapidly evolving libraries
2. Search API references for correct function signatures and usage examples
3. Ensure AI-generated code uses current APIs rather than deprecated or hallucinated methods

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Latest SDK docs for development | ALL | Provides current documentation for any NVIDIA SDK with llms.txt support |

## Prerequisites
- Python 3.8+ with `uvx` (or `pip install mcpdoc`)
- No API key required

## Security Notes
- Documentation is fetched from public endpoints; no sensitive data is transmitted
- Verify llms.txt sources are from official library maintainers to avoid supply-chain risks

## References
- https://github.com/langchain-ai/mcpdoc
- https://llmstxt.org/
