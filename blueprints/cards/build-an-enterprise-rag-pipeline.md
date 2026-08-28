---
name: build-an-enterprise-rag-pipeline
title: "Build an Enterprise RAG Pipeline Blueprint"
description: "Power fast, accurate semantic search across multimodal enterprise data with NVIDIA’s RAG Blueprint—built on NeMo Retriever and Nemotron models—to connect your agents to trusted, authoritative sources of knowledge."
publisher: "nvidia"
type: blueprint
updated: "2026-02-17T19:30:14.812Z"
canonical: https://build.nvidia.com/nvidia/build-an-enterprise-rag-pipeline
source_url: https://build.nvidia.com/qc69jvmznzxy/build-an-enterprise-rag-pipeline.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Build an Enterprise RAG Pipeline Blueprint

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `build-an-enterprise-rag-pipeline` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/build-an-enterprise-rag-pipeline](https://build.nvidia.com/nvidia/build-an-enterprise-rag-pipeline) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/build-an-enterprise-rag-pipeline.md](https://build.nvidia.com/qc69jvmznzxy/build-an-enterprise-rag-pipeline.md) |
| Source snapshot | [../raw/build-an-enterprise-rag-pipeline.md](../raw/build-an-enterprise-rag-pipeline.md) |
| Last updated by publisher | 2026-02-17T19:30:14.812Z |

**Summary:** Power fast, accurate semantic search across multimodal enterprise data with NVIDIA’s RAG Blueprint—built on NeMo Retriever and Nemotron models—to connect your agents to trusted, authoritative sources of knowledge.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

The NVIDIA AI Blueprint for Retrieval-Augmented Generation (RAG) is a production-ready, modular reference architecture for building high-accuracy, high-performance RAG systems that power enterprise search, knowledge assistants, copilots, and agentic workflows at scale. Optimized for GPU acceleration and enterprise throughput, the blueprint provides a complete foundation for ingestion, retrieval, reasoning, and generation across multimodal enterprise data.

Built to support modern agent ecosystems, the blueprint includes shallow and deep document summarization, reasoning-budget configurability, query decomposition, and dynamic metadata filtering—enabling agents to efficiently narrow search space, select trusted sources, and reason over large corpora. Native Python libraries, OpenAI-compatible APIs, MCP server support, and a built-in data catalog make it easy for developers to integrate RAG capabilities into existing applications and multi-agent workflows.

The blueprint supports advanced multimodal generation, including vision-language models (VLMs) for image understanding, captioning, and image-aware answer generation, along with optional reflection to further improve answer quality. A robust multimodal ingestion pipeline extracts text, tables, charts, images, infographics, and audio/video content, enriched with custom metadata to improve downstream retrieval and filtering. S3-compatible object storage is included, with SeaweedFS as the default deployment.

Designed for flexibility and scale, the RAG Blueprint offers hybrid dense + sparse retrieval, multi-collection search, GPU-accelerated indexing and querying, reranking, and pluggable vector database support—including Elasticsearch as the default and Milvus as an optional backend—with fine-grained database authorization and token support. Built-in observability, OpenTelemetry integration, performance tooling, and evaluation scripts (RAGAS) help teams measure accuracy, latency, and quality as they move from pilot to production, while optional programmable guardrails support enterprise safety requirements.

Deployable via Docker, Kubernetes, or Red Hat OpenShift, with a user interface included and support for GPU sharing through the NIM Operator, the blueprint is fully decomposable and customizable to fit domain-specific needs. It can run standalone, integrate with other NVIDIA Blueprints, or serve as a core building block in agentic systems.

Importantly, the NVIDIA AI Blueprint for RAG serves as a foundational layer of the [NVIDIA AI Data Platform](https://www.nvidia.com/en-us/data-center/ai-data-platform/), transforming raw, multimodal enterprise data into AI-ready knowledge that powers retrieval, reasoning, and generation across applications.

It is also foundational to the AI Agent for Enterprise Research, providing the trusted knowledge base, summarization, and retrieval capabilities required for advanced, reasoning-driven enterprise agents [AI Agent for Enterprise Research](https://build.nvidia.com/nvidia/aiq)

Get started with this reference architecture to ground AI-driven decisions and generation in trusted, relevant enterprise data—at production scale.

## Architecture Diagram
![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/build-an-enterprise-rag-pipeline/diagram.jpg)

## Key Features

* Agent Ecosystem Support
* Summarization (Shallow and Deep)
* Agent Skills support
* MCP Server Support
* Data Catalog Support
* Reasoning Budget Configurability
* Native Python library support
* OpenAI-compatible APIs

* Multimodal and Advanced Generation
* Vision Language Model (VLM) Support in answer generation
* Image captioning with vision language models (VLMs)
* Improve accuracy with optional reflection

* Data Ingestion and Processing
* Multimodal PDF data extraction support with text, tables, charts, images and infographics
* Support for audio/video file ingestion
* Custom metadata support
* Continuous Ingestion with object storage
* SeaweedFS object storage by default

* Vector Database and Retrieval
* Agentic RAG pipeline
* Multi-collection searchability
* Hybrid search with dense and sparse search
* Reranking to further improve accuracy
* Optional VLM reranking
* Optional GPU-accelerated index creation and search
* Pluggable vector database
* Elasticsearch support as the default vector database
* Milvus support as an optional vector database
* Query Decomposition
* Dynamic metadata filter generation for Elasticsearch
* Database Authorization Support
* Database Auth token Support

* Governance
* Improve content safety with optional programmable guardrails

* Observability and Telemetry
* Evaluation Scripts included (RAGAS framework)
* Performance benchmarking tooling
* OpenTelemetry Support

* Other
* User interface included
* NIM Operator support to allow GPU sharing
* Red Hat OpenShift support
* Decomposable and customizable
* Multi-turn conversations
* Multi-session support

## Minimum System Requirements

**Hardware Requirements**

The blueprint can be deployed with Docker or Kubernetes-based platforms. By default, it deploys the referenced NIM microservices locally. Each method lists its minimum required hardware. This will change if the deployment turns on optional configuration settings.
* Docker
* 3 x H100
* 3 x B200
* 3 x RTX PRO 6000

* Kubernetes
* 8 x H100-80GB
* 8 x B200
* 8 x RTX PRO 6000
* 5 x H100-80GB (with Multi-Instance GPU)

* Optional GPU-backed services increase the requirement. Plan for one additional GPU for each optional service that you enable, such as VLM generation, VLM captioning, VLM reranking, Nemotron Parse, or audio processing, unless you use MIG slicing or another explicit sharing strategy.
* The blueprint allows for use of NVIDIA-hosted NIM endpoints. Local GPU requirements apply when self-hosting NIM microservices or enabling optional GPU-accelerated vector database features.

**OS Requirements**
* Ubuntu 22.04 OS

**Deployment Options**
* Docker
* Kubernetes with Helm or NIM Operator
* Red Hat OpenShift with Helm

## Software used in this blueprint

**NVIDIA Technology**
* [NVIDIA Nemotron 3 Super 120B A12B](https://build.nvidia.com/nvidia/nemotron-3-super-120b-a12b)
* [Nemotron Llama VLM embedding NIM](https://build.nvidia.com/nvidia/llama-nemotron-embed-vl-1b-v2)
* [Nemotron Llama embedding NIM](https://build.nvidia.com/nvidia/llama-nemotron-embed-1b-v2) *(optional text-only embedder)*
* [Nemotron Llama reranking NIM](https://build.nvidia.com/nvidia/llama-nemotron-rerank-1b-v2)
* [Nemotron Llama VLM reranking NIM](https://build.nvidia.com/nvidia/llama-nemotron-rerank-vl-1b-v2) *(optional)*
* [Nemotron page elements NIM](https://build.nvidia.com/nvidia/nemotron-page-elements-v3)
* [Nemotron table structure NIM](https://build.nvidia.com/nvidia/nemotron-table-structure-v1)
* [Nemotron graphic elements NIM](https://build.nvidia.com/nvidia/nemotron-graphic-elements-v1)
* [Nemotron OCR NIM](https://build.nvidia.com/nvidia/nemotron-ocr-v1)
* [Nemotron Nano Omni 30B A3B Reasoning NIM](https://build.nvidia.com/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning) *(optional VLM generation)*
* [Llama Nemotron Nano 12B V2 VL](https://build.nvidia.com/nvidia/nemotron-nano-12b-v2-vl) *(optional image captioning)*
* [Nemotron parse NIM](https://build.nvidia.com/nvidia/nemotron-parse) *(optional)*
* [Llama 3.1 NemoGuard 8B Content safety NIM](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-content-safety) *(optional)*
* [Llama 3.1 NemoGuard 8B Topic control NIM](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-topic-control) *(optional)*
* [NVIDIA Riva ASR NIM](https://build.nvidia.com/nvidia/parakeet-ctc-1_1b-asr) *(optional)*

**3rd Party Software**
* [LangChain](https://www.langchain.com/)
* [Elasticsearch Vector Database](https://www.elastic.co/elasticsearch/vector-database)
* Milvus database (optional, accelerated with [NVIDIA **cuVS**](https://github.com/rapidsai/cuvs))
* [SeaweedFS object store](https://github.com/seaweedfs/seaweedfs)
* [Redis Cache](https://redis.io/)

## Ethical Considerations
NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## License
Use of the models in this blueprint is governed by the [NVIDIA AI Foundation Models Community License](https://docs.nvidia.com/ai-foundation-models-community-license.pdf).

## Terms of Use

This blueprint is governed by the [NVIDIA Agreements | Enterprise Software | NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/) and the [NVIDIA Agreements | Enterprise Software | Product Specific Terms for AI Product](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/). The models are governed by the [NVIDIA Agreements | Enterprise Software | NVIDIA Community Model License](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-community-models-license/) and the [NVIDIA RAG dataset](https://github.com/NVIDIA-AI-Blueprints/rag/tree/main/data/multimodal) which is governed by the [NVIDIA Asset License Agreement](https://github.com/NVIDIA-AI-Blueprints/rag/blob/main/data/LICENSE.DATA).
The following models that are built with Llama are governed by the [Llama 3.2 Community License Agreement](https://www.llama.com/llama3_2/license/): nvidia/llama-nemotron-embed-1b-v2, nvidia/llama-nemotron-rerank-1b-v2, nvidia/llama-nemotron-embed-vl-1b-v2, and nvidia/llama-nemotron-rerank-vl-1b-v2.
Use of the model NVIDIA Nemotron-3-Super-120B-A12B is governed by the [NVIDIA Nemotron Open Model License.](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-nemotron-open-model-license/)

**ADDITIONAL INFORMATION**:

The [Llama 3.1 Community License Agreement](https://www.llama.com/llama3_1/license/) for the llama-3.1-nemoguard-8b-content-safety and llama-3.1-nemoguard-8b-topic-control models. The [Llama 3.2 Community License Agreement](https://www.llama.com/llama3_2/license/) for the nvidia/llama-nemotron-embed-1b-v2, nvidia/llama-nemotron-rerank-1b-v2, nvidia/llama-nemotron-embed-vl-1b-v2, and nvidia/llama-nemotron-rerank-vl-1b-v2 models. Built with Llama. Apache 2.0 for NVIDIA Ingest and for the nemotron-page-elements-v3, nemotron-table-structure-v1, nemotron-graphic-elements-v1, nemotron-parse, paddleocr, and nemotron-ocr-v1 models.
