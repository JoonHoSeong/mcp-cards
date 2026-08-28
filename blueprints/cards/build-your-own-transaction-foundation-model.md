---
name: build-your-own-transaction-foundation-model
title: "Build Your Own Transaction Foundation Model"
description: "Create intelligent embeddings by using transformer architecture on tabular data."
publisher: "nvidia"
type: blueprint
updated: "2026-04-10T20:49:08.186Z"
canonical: https://build.nvidia.com/nvidia/build-your-own-transaction-foundation-model
source_url: https://build.nvidia.com/qc69jvmznzxy/build-your-own-transaction-foundation-model.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Build Your Own Transaction Foundation Model

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `build-your-own-transaction-foundation-model` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/build-your-own-transaction-foundation-model](https://build.nvidia.com/nvidia/build-your-own-transaction-foundation-model) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/build-your-own-transaction-foundation-model.md](https://build.nvidia.com/qc69jvmznzxy/build-your-own-transaction-foundation-model.md) |
| Source snapshot | [../raw/build-your-own-transaction-foundation-model.md](../raw/build-your-own-transaction-foundation-model.md) |
| Last updated by publisher | 2026-04-10T20:49:08.186Z |

**Summary:** Create intelligent embeddings by using transformer architecture on tabular data.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

The Build Your Own Transaction Foundation Model developer example provides developers with a starting point to build embeddings using transformer architecture on tabular data. 

Transaction foundation models are large-scale AI systems designed to understand, process, and optimize payment transaction related data across diverse domains like transaction authorization, fraud detection, and customer insights. For developers, these models act as a backbone for building intelligent financial tools that can generalize across payment networks, banking institutions, currencies, and compliance rules. By learning from vast datasets of transaction patterns, merchant histories, and financial interactions, they can predict anomalies, automate routing decisions, and personalize user experiences with minimal custom logic. Developers can use APIs or fine-tuning pipelines to adapt these models for specific use cases—like improving checkout conversion or dynamic risk scoring—without starting from scratch. In essence, transaction foundation models transform raw financial data into actionable intelligence, dramatically accelerating innovation in fintech systems. These models find use in payments, banking, capital markets, and the principles apply broadly to any industry with tabular datasets.

This developer example provides a reference example to build transformer embeddings on tabular data. Such embeddings, when fed as features to machine learning models, are a powerful means to boost accuracy or reduce false positives on top of existing models. The interconnected nature of these embeddings ensure that they cater to a variety of use cases. The developer example leverages NVIDIA CUDA-X libraries for data processing, and NVIDIA NeMo framework for model fine-tuning.

## Architecture Diagram
![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/build-your-own-transaction-foundation-model/diagram.jpg)

## Key Features

* Ease of creating embeddings \- ready to use recipe  
* Hands on techniques to showcase the process of tokenizing tabular transactions  
* Leveraging the best of Nemo framework capabilities to build these embeddings  
* Training at scale using Nemo Automodel library  
* Optimized for speed and performance at scale   
* Ability to augment existing machine learning models with new embeddings based features

## Minimum System Requirements

**Hardware Requirements**

* GPU: 1x NVIDIA A100 (80GB) or H100  
* RAM: 32 GB  
* OS: Ubuntu 22.04

**Deployment Options**

* Docker

## Software used in this blueprint

**NVIDIA Technology**

* [NVIDIA NeMo](https://www.nvidia.com/en-us/ai-data-science/products/nemo/) (AutoModel) – Foundation model training and inference  
* [NVIDIA RAPIDS™](https://developer.nvidia.com/rapids) (cuDF, cuML) – GPU-accelerated data processing and tokenization

**3rd Party Software**

* PyTorch 2.x — Deep learning framework  
* HuggingFace Transformers — Model checkpointing and loading  
* XGBoost — Gradient-boosted trees for fraud detection  
* scikit-learn — Classical ML preprocessing, metrics, and baseline utilities  
* pandas — CPU dataframe operations and interoperability with GPU pipelines  
* NumPy — Array operations used across preprocessing and inference  
* CuPy — GPU array operations for tokenizer and embedding workflows  
* matplotlib — Static visualizations  
* seaborn — Statistical plotting for dataset exploration  
* plotly — Interactive 3D embedding visualization  
* tqdm — Progress bars in notebook inference workflows  
* ipywidgets — Notebook widget support  
* torchdata — Stateful data loading for model training

## License

By using this software, you are agreeing to the terms and conditions of the license and acceptable use policy.

#### Terms of Use

GOVERNING TERMS: This developer example is governed by the [NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/) and [Product Specific Terms for AI Products](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/).

This project will download and install additional third-party open source software projects. Review the license terms of these open source projects before use.

Please report security vulnerabilities or NVIDIA AI concerns [here](https://app.intigriti.com/programs/nvidia/nvidiavdp/detail).
