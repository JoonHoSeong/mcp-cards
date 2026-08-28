---
name: financial-fraud-detection
title: "Financial Fraud Detection"
description: "Detect and prevent sophisticated fraudulent activities for financial services with high accuracy."
publisher: "nvidia"
type: blueprint
updated: "2026-02-17T19:23:48.772Z"
canonical: https://build.nvidia.com/nvidia/financial-fraud-detection
source_url: https://build.nvidia.com/qc69jvmznzxy/financial-fraud-detection.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Financial Fraud Detection

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `financial-fraud-detection` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/financial-fraud-detection](https://build.nvidia.com/nvidia/financial-fraud-detection) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/financial-fraud-detection.md](https://build.nvidia.com/qc69jvmznzxy/financial-fraud-detection.md) |
| Source snapshot | [../raw/financial-fraud-detection.md](../raw/financial-fraud-detection.md) |
| Last updated by publisher | 2026-02-17T19:23:48.772Z |

**Summary:** Detect and prevent sophisticated fraudulent activities for financial services with high accuracy.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

Financial losses from worldwide credit card transaction fraud are [projected](https://www.paymentsdive.com/news/payments-fraud-losses-prevention-nilson-outlook/737440/) to reach more than $403 billion over the next decade. Transaction fraud poses a major challenge for financial institutions, which struggle to detect and prevent increasingly complicated fraudulent activities. Traditional fraud detection methods, which rely on rules-based systems or statistical methods, are reactive and increasingly ineffective in identifying sophisticated fraudulent activities. As data volumes grow and fraud tactics evolve, financial institutions need more proactive, intelligent approaches to detect and prevent fraudulent transactions.

This NVIDIA AI Blueprint provides a reference example to detect and prevent sophisticated fraudulent activities for financial services with high accuracy and reduced false positives. It shows developers how to build a financial fraud detection workflow using the NVIDIA container for fraud detection. For model building, the Financial Fraud Training container augments fraud detection using graph neural networks (GNNs)—a deep learning technique—for improved accuracy. Inference is done using the NVIDIA Dynamo-Triton (formerly Triton Inference Server) and produces fraud scores along with Shapley values for explainability. Furthermore, to help simplify the workflow, the Financial Fraud Training container also produces all the needed configuration files required by Dynamo-Triton.  

## Architecture Diagram
This NVIDIA AI blueprint is broken down into three steps, which map to processes within a typical payment processing environment, those steps being: (1) Data Preparation, (2) Model Building, and (3) Inference. Additionally, within a production system, the event data would most likely be saved within a database or a data lake.  For this example, the data is just a collection of files with synthetic data.

![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/financial-fraud-detection/diagram.jpg)

## What’s Included in the Blueprint

**NVIDIA Technology**

* [The Financial Fraud Training container](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/cugraph/containers/financial-fraud-training)  
* [NVIDIA Dynamo-Triton](https://developer.nvidia.com/triton-inference-server) (formerly Triton Inference Server)  
* [NVIDIA RAPIDS™](https://developer.nvidia.com/rapids) for data preparation

## Key Features
This NVIDIA AI blueprint provides an example and documentation of using the NVIDIA container for financial fraud detection. The container is designed to be used in a payments workflow looking to integrate capabilities for catching fraud signals. Included in this workflow are two key operational components representative of common activities one might see in a production system, namely model building and inference.

__Model Building__

Model building is a key component for fraud detection, which takes cleaned and prepared label data and produces a model that can be used to predict fraudulent scores in financial payment events. The Financial Fraud Training container leverages a graph neural network (GNN) to generate embeddings that are then fed into XGBoost to produce the model.

The goal in a production environment is to produce new models and update existing models as often as possible when new data is ingested. Frequent creation and updates of a new model helps identify new and evolving fraudulent activities. The model building process can be complex, with data being split across different types of data structures and technologies.

The benefits of using the Financial Fraud Training container are:

* **No GNN Expertise Required:** Empower data scientists and engineers to build advanced models without needing deep knowledge of GNNs.  
* **Streamlined Model Pipeline:** Automates the configuration and execution of the full GNN \+ XGBoost model pipeline—eliminating complexity and saving development time.  
* **Seamless Inference Integration:** Generates all necessary configuration files for direct deployment into Dynamo-Triton, simplifying the path to production.  
* **Optimized for Speed by Default:** Always selects the fastest processing framework available, ensuring high-performance model execution with no manual tuning.  
* **Effortless Upgrades:** Stay current with zero hassle—updating to the latest version is as easy as pulling the newest container image.

It is worth noting that data preparation is a critical initial process that needs to be done. Poorly prepared data will lead to poor accuracy. This blueprint provides an example of data formatting.    

__Inference__

Inference is the process of predicting a fraudulent score for each input event record. The process does not flag an event as fraudulent; it simply provides a score that the system can then use to determine fraud. For inference, the blueprint leverages the Dynamo-Triton. The Financial Fraud Training container produces the model and all the configuration files needed by Dynamo-Triton.

## Minimum System Requirements

### Hardware Requirements
* GPU: 1x A6000, A100, or H100, minimum of 32 GB of memory   
* CPU: x86\_64 architecture  
* Storage: 10 GB  
* System Memory: 16 GB

### Software Requirements
* Operating System: Ubuntu 20.04 or newer  
* NVIDIA Driver version: 535 or newer  
* NVIDIA CUDA version: 12.4 or newer  
* NVIDIA Container Toolkit version: 1.15.0 or newer
* Docker version: Docker version 26 or newer

## Ethical Considerations

NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy Subcards. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## License

Use of the models in this blueprint is governed by the [NVIDIA AI Foundation Models Community License](https://docs.nvidia.com/ai-foundation-models-community-license.pdf).

## Terms of Use
The software and materials are governed by the [NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/) and the [Product-Specific Terms for NVIDIA AI Products](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/) , except that models are governed by the [AI Foundation Models Community License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-community-models-license/) and the [NVIDIA RAG dataset](https://github.com/NVIDIA-AI-Blueprints/rag/blob/main/data/LICENSE.DATA) is governed by the NVIDIA Asset License Agreement. 

Additional Information: for Meta/llama-3.1-70b-instruct model the Llama 3.1 Community License Agreement, for nvidia/llama-3.2-nv-embedqa-1b-v2model the Llama 3.2 Community License Agreement, and for nvidia/llama-3.2-nv-embedqa-1b-v2 model the Llama 3.2 Community License Agreement. Built with Llama.
