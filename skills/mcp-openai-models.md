# OpenAI Models MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-server-openai |
| **Category** | AI/ML |
| **Official** | Community |
| **Source** | https://github.com/pierrebrunelle/mcp-server-openai |
| **Transport** | stdio |
| **Install** | `npx mcp-server-openai` |

## Description
OpenAI MCP Server enables cross-model orchestration by giving AI agents (such as Claude) direct access to OpenAI's model capabilities. It supports chat completions, embeddings, image generation, audio transcription, and text-to-speech — allowing workflows to leverage the best model for each subtask regardless of the host LLM, enabling multi-model evaluation and comparison pipelines.

## Key Tools
| Tool | Description |
|------|-------------|
| `chat_completion` | Generate chat completions using GPT models |
| `list_models` | List all available OpenAI models |
| `create_embedding` | Generate text embeddings for semantic search |
| `create_image` | Generate images using DALL-E models |
| `transcribe_audio` | Transcribe audio files using Whisper |
| `text_to_speech` | Convert text to speech audio |

## Configuration
```json
{
  "mcpServers": {
    "openai": {
      "command": "npx",
      "args": ["mcp-server-openai"],
      "env": {
        "OPENAI_API_KEY": "sk-your_key_here"
      }
    }
  }
}
```

## Use Cases
1. Run multi-model evaluations comparing GPT outputs against other LLMs from a single agent
2. Generate embeddings for RAG pipelines using OpenAI's embedding models
3. Orchestrate multimodal workflows combining vision, audio, and text generation

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Compare TTS outputs | nemotron-speech | Benchmark OpenAI TTS against NVIDIA Nemotron speech synthesis |
| Multi-model evaluation | rag-eval | Evaluate RAG quality using multiple model backends for consensus |

## Prerequisites
- `OPENAI_API_KEY` from https://platform.openai.com/api-keys
- Node.js 18+ (for npx)

## Security Notes
- API key grants access to all OpenAI services including billing; use project-scoped keys
- Community-maintained server — audit source code before use in production environments
- Input data is sent to OpenAI's servers; ensure compliance with data policies

## References
- https://github.com/pierrebrunelle/mcp-server-openai
- https://platform.openai.com/docs/api-reference
