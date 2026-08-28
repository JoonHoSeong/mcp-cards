---
name: genomics-analysis
title: "Genomic Analysis"
description: "Easily run essential genomics workflows to save time leveraging Parabricks and CodonFM."
publisher: "nvidia"
type: blueprint
updated: "2026-02-17T19:26:24.636Z"
canonical: https://build.nvidia.com/nvidia/genomics-analysis
source_url: https://build.nvidia.com/qc69jvmznzxy/genomics-analysis.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Genomic Analysis

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `genomics-analysis` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/genomics-analysis](https://build.nvidia.com/nvidia/genomics-analysis) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/genomics-analysis.md](https://build.nvidia.com/qc69jvmznzxy/genomics-analysis.md) |
| Source snapshot | [../raw/genomics-analysis.md](../raw/genomics-analysis.md) |
| Last updated by publisher | 2026-02-17T19:26:24.636Z |

**Summary:** Easily run essential genomics workflows to save time leveraging Parabricks and CodonFM.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

## Overview

This developer example enables bioinformaticians to run GPU-accelerated genomics workflows in minutes on any cloud through Brev.dev. [NVIDIA® Parabricks®](https://docs.nvidia.com/clara/parabricks/latest/index.html) powers both linear and graph-based read alignment along with variant calling via DeepVariant. [CodonFM](https://github.com/NVIDIA-Digital-Bio/CodonFM), NVIDIA's open-source suite of foundation models for RNA codon sequences, can then be used to predict the functional impact of each detected variant on specific genes. 

## Experience Workflow

This developer example shows how to use GPU accelerated tools for alignment (linear and graph), variant calling, and variant effect prediction. 

### Architecture Diagram

The exact steps to run this workflow are outlined below: 

![](https://assets.ngc.nvidia.com/products/api-catalog/genomics-analysis/diagram.jpg)

### Notebook Outline 

All the code can be found in Jupyter notebooks in the [`notebooks`](https://github.com/NVIDIA-AI-Blueprints/genomics-analysis/notebooks) directory of the Github repo. 

#### `germline_wes.ipynb`
Runs a standard germline variant calling workflow on whole exome sequencing (WES) data using NVIDIA Parabricks. Downloads the NA12878 sample from the Genome in a Bottle consortium, aligns reads to the GRCh38 reference using GPU-accelerated BWA-MEM via Parabricks fq2bam, and calls variants with GPU-accelerated DeepVariant, producing a final .vcf file.

#### `pangenome.ipynb`
Demonstrates a pangenome analysis workflow as an alternative to single-reference alignment using NVIDIA Parabricks. Downloads the HPRC v1.1 pangenome graph, aligns short-read FASTQ samples using GPU-accelerated Giraffe, and calls variants with Pangenome-Aware DeepVariant — a variant of DeepVariant that uses the pangenome graph to improve alignment accuracy and variant detection across diverse populations.

#### `variant_effect_prediction.ipynb`
Runs a full variant effect prediction pipeline starting from raw FASTQ files. It uses NVIDIA Parabricks to align reads and call variants, processes GENCODE gene annotations to extract protein-coding sequences, maps detected variants onto transcripts, and uses CodonFM (NVIDIA's RNA foundation model) to predict the functional impact of each variant via log likelihood ratios.

## How to Run 

### Hardware Requirements 

The L40s with at least 48GB of GPU memory is recommended for the best combination of cost and performance. Users can also try L4 or T4 (better cost) or RTX Pro 6000 (better performance). 

NVIDIA Parabricks can be run on any NVIDIA GPU that supports CUDA® architecture 75, 80, 86, 89, 90, 100, or 120 and has at least 16GB of GPU RAM. 

Parabricks has been tested specifically on the following NVIDIA GPUs:  

* T4  
* A10, A30, A40, A100, A6000  
* L4, L40  
* H100, H200  
* GH200
* B200, B300
* GB200, GB300
* RTX PRO 6000 Blackwell Server Edition
* RTX PRO 4500
* DGX Spark
* DGX Station

The minimum amount of CPU RAM and CPU threads depends on the number of GPUs. Please refer to the table below: 

| GPUs | Minimum CPU RAM (GB) | Minimum CPU Threads |
|------|----------------|---------------------|
| 2 | 100 | 24 |
| 4 | 196 | 32 |
| 8 | 392 | 48 |

### Software Requirements 

* Any NVIDIA driver that is compatible with CUDA 12.9 (535, 550, 570, 575, or similar). Please check [here](https://docs.nvidia.com/deploy/cuda-compatibility/#forward-compatibility) for more details on forward compatibility.  
* Any Linux operating system that supports Docker version 20.10 (or higher) with the NVIDIA GPU runtime.

## References

* [Parabricks Documentation](https://docs.nvidia.com/clara/parabricks/latest/index.html) 
* [Parabricks Pangenome Alignment Blog](https://developer.nvidia.com/blog/discover-new-biological-insights-with-accelerated-pangenome-alignment-in-nvidia-parabricks/)
* [CodonFM Blog](https://developer.nvidia.com/blog/introducing-the-codonfm-open-model-for-rna-design-and-analysis/)

## Terms of Use

**Governing Terms**: The Blueprint scripts are governed by the [NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/), the [Product-Specific Terms for NVIDIA AI Products](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/), and enables use of separate open source and proprietary software governed by their respective licenses: 

* [NVIDIA Parabricks](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/clara/containers/clara-parabricks?version=4.6.0-2) ([NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/), the [Product-Specific Terms for NVIDIA AI Products](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/))
* [NV-CodonFM-Encodon-TE-80M-v1](https://huggingface.co/nvidia/NV-CodonFM-Encodon-TE-80M-v1) model ([NVIDIA Open Model License](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-license/)); 
* [CodonFM](https://github.com/NVIDIA-Digital-Bio/CodonFM) ([Apache 2.0](https://github.com/NVIDIA-Digital-Bio/CodonFM?tab=Apache-2.0-1-ov-file#readme)); and 
* The other supporting open source software in the [Clara Parabricks Workflows public repo](https://github.com/clara-parabricks-workflows) are governed by their accompanying licenses ([Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0.txt), [BSD 3-Clause](https://opensource.org/license/bsd-3-clause), [MIT License](https://opensource.org/license/mit), and [PIGZ License](https://wonder.cdc.gov/amd/flu/irma/pigz_license.html)). 

## Ethical Considerations

NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and addresses unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy Subcards. Please report security vulnerabilities or NVIDIA AI Concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).
