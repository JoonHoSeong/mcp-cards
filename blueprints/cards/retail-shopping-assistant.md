---
name: retail-shopping-assistant
title: "Retail Shopping Assistant"
description: "Elevate Shopping Experiences Online and In Stores."
publisher: "nvidia"
type: blueprint
updated: "2026-02-17T19:31:28.303Z"
canonical: https://build.nvidia.com/nvidia/retail-shopping-assistant
source_url: https://build.nvidia.com/qc69jvmznzxy/retail-shopping-assistant.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Retail Shopping Assistant

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `retail-shopping-assistant` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/retail-shopping-assistant](https://build.nvidia.com/nvidia/retail-shopping-assistant) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/retail-shopping-assistant.md](https://build.nvidia.com/qc69jvmznzxy/retail-shopping-assistant.md) |
| Source snapshot | [../raw/retail-shopping-assistant.md](../raw/retail-shopping-assistant.md) |
| Last updated by publisher | 2026-02-17T19:31:28.303Z |

**Summary:** Elevate Shopping Experiences Online and In Stores.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

AI-powered shopping assistants give retailers a new way to engage customers — intelligently, personally, and at any hour across any market.

Today's shoppers expect more than keyword search. They arrive with complex, open-ended needs: outfitting a backyard, planning a themed birthday party, or finding the right combination of products for a project they can barely describe. Existing tools fall short here — built for short, structured queries, they leave customers empty-handed and retailers without a sale. Every failed search is a missed opportunity to convert, upsell, or build loyalty.

This NVIDIA AI Blueprint gives retailers a production-ready reference for deploying a shopping assistant that understands natural language and images, handles multi-step discovery, and delivers personalized, context-aware recommendations — driving higher conversion rates, larger order sizes, and fewer returns.

## Architecture Diagram
![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/retail-shopping-assistant/diagram.jpg)

## Key Features

* An end-to-end sample multimodal, multi-query agentic pipeline with image-aware query routing and image-to-image similarity search via NVClip, enabling consumers to use text and images together in natural queries
* Optimized LLM inference performance and scaling through NIM microservices, bringing advanced reasoning capabilities for natural, humanlike interactions
* Intelligent cart management with natural language add, remove, and update commands — including persistent price tracking and pronoun resolution across multi-turn conversations
* Language-agnostic product matching with catalog-derived vocabulary, making the assistant portable across non-English retail catalogs without additional configuration
* Guardrails that keep customer conversations safe and on-topic through dual NeMo Guard NIM microservices for content safety and topic control, protecting brand values
* High accuracy product discovery and data privacy powered by NVIDIA Retrieval QA E5 Embedding v5
* Integration with LangChain and the NVIDIA cuVS GPU-accelerated Milvus vector database
* Sample retail product catalog and imagery with the ability to ingest retailers' own product catalog text and image data for accurate, context-aware responses
* Production-ready test infrastructure with unit and integration coverage across all services and CI/CD workflows for automated validation
* The flexibility to use other models from the NVIDIA API catalog or self-hosted models

![Screenshot image](https://assets.ngc.nvidia.com/products/api-catalog/retail-shopping-assistant/screenshot1.jpg)

## Minimum System Requirements

### Hardware Requirements
* 4 x H100 (Locally hosted models)

### Deployment Options
* Docker

## Software Used in This Blueprint

**NVIDIA Technology**

* [NVIDIA Retrieval QA E5 Embedding v5](https://build.nvidia.com/nvidia/nv-embedqa-e5-v5)
* [NVclip](https://build.nvidia.com/nvidia/nvclip)
* [Nemotron 3 super 120b a12b NIM](https://build.nvidia.com/nvidia/nemotron-3-super-120b-a12b)
* [Llama 3.1 Nemoguard 8B Topic Control](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-topic-control)
* [Llama 3.1 Nemoguard 8B Content Safety](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-content-safety)

**3rd Party Software**

* [LangChain](https://www.langchain.com/)
* Milvus database (accelerated with NVIDIA [**cuVS**](https://github.com/rapidsai/cuvs))
* SQLite

## Ethical Considerations
NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy Subcards. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## License
GOVERNING TERMS: Use of the blueprint software and materials and NIM containers are governed by the [NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/) and [Product-specific Terms for AI products](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/);  and the use of models is governed by the [NVIDIA Community Model License](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-community-models-license/).

ADDITIONAL INFORMATION: [Llama 3.1 Community License Agreement](https://www.llama.com/llama3_1/license/) for Llama 3.1 70B Instruct NIM, Llama 3.1 NemoGuard 8B \- Content Safety and Llama 3.1 NemoGuard 8B \- Topic Control models, built with Llama, (ii) MIT license for NV-EmbedQA-E5-v5.

Use of the product catalog data in the retail shopping assistant is governed by the terms of the [NVIDIA Data License for Retail Shopping Assistant (15Aug2025)](https://github.com/NVIDIA-AI-Blueprints/retail-shopping-assistant/blob/main/LICENSE-assets.txt).
