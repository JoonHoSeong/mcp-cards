---
name: aiq
title: "NVIDIA AI-Q Blueprint for intelligent agents"
description: "AI agents that connect, retrieve, and reason on enterprise data—making information accessible, actionable, and intelligent."
publisher: "nvidia"
type: blueprint
updated: "2026-02-13T18:54:18.718Z"
canonical: https://build.nvidia.com/nvidia/aiq
source_url: https://build.nvidia.com/qc69jvmznzxy/aiq.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# NVIDIA AI-Q Blueprint for intelligent agents

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `aiq` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/aiq](https://build.nvidia.com/nvidia/aiq) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/aiq.md](https://build.nvidia.com/qc69jvmznzxy/aiq.md) |
| Source snapshot | [../raw/aiq.md](../raw/aiq.md) |
| Last updated by publisher | 2026-02-13T18:54:18.718Z |

**Summary:** AI agents that connect, retrieve, and reason on enterprise data—making information accessible, actionable, and intelligent.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

## NVIDIA AI-Q Blueprint

The NVIDIA AI-Q Blueprint (pronounced IQ) is a deployable, customizable research system built on [LangChain Deep Agents](https://docs.langchain.com/oss/python/deepagents/overview) and accelerated by the [NVIDIA NeMo Agent Toolkit](https://docs.nvidia.com/nemo/agent-toolkit/latest/). Teams can self-host the application boundary and connect deployment-owned models, enterprise data, authentication, policy controls, storage, and observability. AI-Q combines fast, cited answers with in-depth, report-style research and includes evaluation harnesses for measuring quality.

## Architecture

![AI-Q Architecture](https://assets.ngc.nvidia.com/products/api-catalog/aiq/diagram1.jpg)

Every query enters through an intent classifier, which responds directly to conversational requests or routes research to a shallow or deep path. Deep research can clarify the request, consult an optional source router, build a structured plan, dispatch concurrent researcher workers, and delegate final synthesis to a writer. Research roles share job-scoped state; configured skills can execute code in an isolated [NVIDIA OpenShell](https://docs.nvidia.com/openshell/latest/about/installation) or Modal sandbox without moving inference, source credentials, or enterprise data out of the AI-Q process.

## Key Features

AI-Q is powered by a LangGraph-based state machine. The agents can run as one orchestrated research workflow or as standalone components:

- **Orchestration node**: Classifies intent, produces conversational responses when appropriate, and routes research to the shallow or deep path.
- **Shallow research agent**: Performs bounded, tool-augmented research optimized for fast, cited answers.
- **Deep research agent**: Coordinates optional source routing, structured planning, concurrent researcher workers, and writer-led synthesis for report-style research.
- **Report follow-up**: Answers questions about a completed report, creates rewrites, or performs additional research with the report as context.
- **Workflow configuration**: YAML profiles define agents, tools, models, source selection, policies, and execution behavior without code changes.
- **Modular workflows**: The orchestration node, shallow researcher, deep researcher, clarifier, and deep-research roles are composable within the full pipeline.
- **Pluggable data sources**: Connect web and paper search, MCP tools, collaboration services, LlamaIndex, the NVIDIA RAG Blueprint, Azure AI Search, and OpenSearch.
- **MCP integration**: Connect to MCP servers through NeMo Agent Toolkit or expose AI-Q research operations through the standalone MCP server.
- **Skills and sandbox execution**: Assign reusable skills to research and writing roles and run generated code in a job-scoped OpenShell or Modal sandbox.
- **Durable generated files**: Capture generated charts, CSVs, notebooks, and documents in SQL or S3-compatible storage for live and replayed access in the UI.
- **Evaluation harnesses**: Use built-in FreshQA and DeepResearch evaluation workflows to measure quality and iterate on prompts and agent architecture.
- **Frontend options**: Run through the CLI, web UI, or asynchronous jobs API.
- **Deployment options**: Self-host with Docker Compose or Helm and connect deployment-owned models, databases, object storage, authentication, policy controls, and observability.

## Prerequisites

**Required:**

- Python 3.11-3.13
- [uv](https://github.com/astral-sh/uv) package manager
- Node.js 22+ and npm (optional, for web UI mode)
- API key for your chosen provider(s):
- NVIDIA API key from [build.nvidia.com](https://build.nvidia.com/) (for NVIDIA NIM inference microservices)
- OpenAI API key (for OpenAI models)
- Anthropic API key (for Claude models)
- Google API key (for Gemini models)

**Optional:**

- API credentials for the research sources and enterprise services enabled by your selected workflow

## System Requirements

**Local / Hybrid Development**

- Developer machine to run the AI-Q instance (no local GPU required)
- LlamaIndex (optional local RAG)
- Provider, service, or RAG APIs

**Fully Self-Hosted / On-Prem**

- Server for AI-Q instances
- [NVIDIA Nemotron 3.5 Lightning 30B A3B](https://build.nvidia.com/nvidia/nemotron-3.5-lightning-30b-a3b) (intent classification and shallow research)
- [NVIDIA Nemotron 3 Ultra 550B A55B](https://build.nvidia.com/nvidia/nemotron-3-ultra-550b-a55b) (clarification and deep-research roles)
- [Google Gemma 4 31B IT](https://build.nvidia.com/google/gemma-4-31b-it) (optional document summary)
- [NVIDIA RAG Blueprint](https://github.com/NVIDIA-AI-Blueprints/rag/blob/main/docs/support-matrix.md) (optional RAG)
- Models packaged as NVIDIA NIM microservices:
- [Llama 3.3 Nemotron Super 49B v1.5](https://build.nvidia.com/nvidia/llama-3_3-nemotron-super-49b-v1_5)
- [Llama 3.2 NV EmbedQA 1B v2](https://build.nvidia.com/nvidia/llama-3_2-nv-embedqa-1b-v2)
- [Llama 3.2 NV RerankQA 1B v2](https://build.nvidia.com/nvidia/llama-3_2-nv-rerankqa-1b-v2)
- [NeMo Retriever Page Elements v3](https://build.nvidia.com/nvidia/nemoretriever-page-elements-v3)
- [NeMo Retriever Table Structure v1](https://build.nvidia.com/nvidia/nemoretriever-table-structure-v1)
- [NeMo Retriever Graphic Elements v1](https://build.nvidia.com/nvidia/nemoretriever-graphic-elements-v1)
- [NeMo Retriever OCR](https://build.nvidia.com/nvidia/nemoretriever-ocr)
- LlamaIndex (optional RAG)
- [NVIDIA Nemotron 3 Embed 1B](https://build.nvidia.com/nvidia/nemotron-3-embed-1b)
- [NVIDIA Nemotron 3 Nano Omni 30B A3B Reasoning](https://build.nvidia.com/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning)

**Hosted Service**

- Server for AI-Q instances
- Provider APIs
- [NVIDIA RAG Blueprint](https://github.com/NVIDIA-AI-Blueprints/rag/blob/main/docs/support-matrix.md) (optional)
- LlamaIndex (optional)

The models above correspond to the default AI-Q profiles. Other checked-in profiles can use different hosted model providers; no single profile enables every capability.

## Hardware Requirements

Hardware requirements vary by model profile, concurrency, context length, and retrieval deployment. Refer to the following resources before sizing a self-hosted deployment:

- [NVIDIA Nemotron 3.5 Lightning model card](https://build.nvidia.com/nvidia/nemotron-3.5-lightning-30b-a3b/modelcard)
- [NVIDIA Nemotron 3 Ultra model card](https://build.nvidia.com/nvidia/nemotron-3-ultra-550b-a55b/modelcard)
- [Google Gemma 4 31B IT model card](https://build.nvidia.com/google/gemma-4-31b-it/modelcard)
- [NVIDIA Nemotron embedding support matrix](https://docs.nvidia.com/nim/nemo-retriever/text-embedding/latest/support-matrix.html)
- [NVIDIA vision-language model support matrix](https://docs.nvidia.com/nim/vision-language-models/latest/support-matrix.html)
- [NVIDIA RAG Blueprint support matrix](https://github.com/NVIDIA-AI-Blueprints/rag/blob/main/docs/support-matrix.md)

## Software Components

**NVIDIA Technology**

- [NVIDIA NeMo Agent Toolkit](https://github.com/NVIDIA/NeMo-Agent-Toolkit)
- [NVIDIA NeMo Guardrails](https://docs.nvidia.com/nemo/guardrails/latest/)
- [NVIDIA NIM](https://build.nvidia.com/models)
- [NVIDIA RAG Blueprint](https://github.com/NVIDIA-AI-Blueprints/rag)

**3rd Party Software**

- [LangChain](https://www.langchain.com/) and [LangGraph](https://www.langchain.com/langgraph) for agent workflows
- [Tavily](https://tavily.com/), [Exa](https://exa.ai/), [You.com](https://you.com/), and [Nimble](https://docs.nimbleway.com/) for configurable web research
- [LlamaIndex](https://www.llamaindex.ai/), Azure AI Search, and OpenSearch for enterprise retrieval

## License

This project is licensed under the Apache License 2.0. See the [AI-Q license](https://github.com/NVIDIA-AI-Blueprints/aiq/blob/release/2.2/LICENSE) for details.

## Ethical Considerations

NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy Subcards. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## Terms of Use

This service is governed by the [NVIDIA API Trial Terms of Service](https://assets.ngc.nvidia.com/products/api-catalog/legal/NVIDIA%20API%20Trial%20Terms%20of%20Service.pdf).
