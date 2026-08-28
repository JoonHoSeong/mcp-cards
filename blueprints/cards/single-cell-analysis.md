---
name: single-cell-analysis
title: "Single Cell Analysis"
description: "Investigate, understand, and interpret single-cell data in minutes, not days, by leveraging RAPIDS-singlecell, powered by NVIDIA CUDA-X Data Science (RAPIDS™)."
publisher: "nvidia"
type: blueprint
updated: "2026-02-18T23:03:32.121Z"
canonical: https://build.nvidia.com/nvidia/single-cell-analysis
source_url: https://build.nvidia.com/qc69jvmznzxy/single-cell-analysis.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Single Cell Analysis

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `single-cell-analysis` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/single-cell-analysis](https://build.nvidia.com/nvidia/single-cell-analysis) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/single-cell-analysis.md](https://build.nvidia.com/qc69jvmznzxy/single-cell-analysis.md) |
| Source snapshot | [../raw/single-cell-analysis.md](../raw/single-cell-analysis.md) |
| Last updated by publisher | 2026-02-18T23:03:32.121Z |

**Summary:** Investigate, understand, and interpret single-cell data in minutes, not days, by leveraging RAPIDS-singlecell, powered by NVIDIA CUDA-X Data Science (RAPIDS™).

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

## Overview

For single-cell analysis, scientists can test near-real-time data analysis and visualization easily, achieving up to 938X faster accelerations versus CPU by using [RAPIDS-singlecell](https://rapids-singlecell.readthedocs.io/), developed by [scverse](https://scverse.org/about/). This blueprint is for scientists who understand single-cell analysis and want to leverage [RAPIDS](https://rapids.ai/) for single-cell data.

## Experience Workflow

It is strongly recommended that users review the [README](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint/blob/main/README.md) in this blueprint before working through the notebooks.

For this blueprint, two possible deployments are provided: 

1. The Standard Instance: 1x L40s  
2. The Advanced Instance: 2x RTX Pro 6000

Please use the table in the Notebook Overview below to determine which size is right for you.

The workflow is as follows:

1. After initial code setup, this blueprint utilizes publicly available datasets including those from [10x Genomics](https://www.10xgenomics.com/datasets) and [CZ CELLxGENE](https://cellxgene.cziscience.com/). Scientists can use their [Python API](https://chanzuckerberg.github.io/cellxgene-census/python-api.html#) to read the data directly into an [AnnData](https://anndata.readthedocs.io/en/stable/) object.  
2. General data preprocessing is performed to clean up and better understand the dataset. This includes calculating QC metrics, filtering, and data normalization.  
3. The data is investigated quantitatively and visually, including feature selection, clustering, dimensionality reduction, and data integration using canonical tools.  
4. The data is visualized and plotted to help users investigate the biological diversity within the sample.  
5. A number of additional advanced tutorials are available for users who are interested in perturbation analysis, spatial transcriptomics analysis, as well as scaling to 11M cells easily and quickly. 

### **Notebooks Outline**

The first three notebooks are an introduction to RAPIDS-singlecell and use the Standard Instance. Unless otherwise noted, users can choose any advanced notebook to get started, as long as the GPU resources are available to run the notebook.

For those who are new to doing basic analysis for single-cell data, the end-to-end analysis of [01_scRNA_analysis_preprocessing](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint/blob/main/notebooks/01_scRNA_analysis_preprocessing.ipynb) is the best place to start, where users are walked through the steps of data preprocessing, cleanup, visualization, and investigation.

**`01_scRNA_analysis_preprocessing.ipynb`** - *Standard Instance* <br>
End to end workflow, where we understand the cells, run ETL on the data set then visiualize and explore the results. This tutorial is good for all users.

**`02_scRNA_analysis_extended.ipynb`** - *Standard Instance* <br>
This notebook continues from the outputs of 01_scRNA_analysis_preprocessing.ipynb as an overview of methods that can be used to investigate transcriptional regulation.

**`03_scRNA_analysis_with_pearson_residuals.ipynb`** - *Standard Instance* <br>
End to end workflow, like 01_scRNA_analysis_preprocessing.ipynb, but uses pearson residuals for normalization.

**`04_scRNA_analysis_dask_out_of_core.ipynb`** - *Advanced Instance* <br>
In this notebook, we show the scalability of the analysis to up to 11M cells easily by using Dask and out of core processing.

**`05_spatial_demo.ipynb`** - *Advanced Instance* <br>
GPU-accelerated spatial analysis using rapids-singlecell and Squidpy. Covers spatial autocorrelation (Moran's I and Geary's C) and co-occurrence analysis to reveal cell-type co-localization and tissue organization patterns.

**`06_scRNA_analysis_1.0M_brain_example.ipynb`** - *Advanced Instance* <br>
In this notebook, we scale up the analysis of the 01_scRNA_analysis_preprocessing.ipynb example to 1 million brain cells.

**`07_perturbation_analysis_invivo_brain_example.ipynb`** - *Advanced Instance* <br>
GPU-accelerated perturbation analysis on a whole-brain single-nucleus CRISPR atlas (~4.2M cells, ~2,000 target genes). Computes pairwise E-distances between perturbation groups and non-targeting controls across neuronal cell types to build a global perturbation-response map.

## Architecture Diagram

## ![](https://assets.ngc.nvidia.com/products/api-catalog/single-cell-analysis/diagram.jpg)

## Software

The following containers are used in this blueprint:

* [RAPIDS](https://developer.nvidia.com/rapids) v26.02

Additional software—including use of [RAPIDS-singlecell](https://rapids-singlecell.readthedocs.io/en/latest/), developed by [scverse](https://scverse.org/about/)—[is available on GitHub accompanying these notebooks](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint/tree/main).

## Minimum System Requirements

On Brev, users may have to wait 5–10 minutes for the instance to start, depending on cloud availability.

### Hardware Requirements

| Instance | GPU Memory | Recommended GPU | RAPIDS Version | CUDA Version |
|---|---|---|---|---|
| **Standard Instance** | >24 GB | 1x NVIDIA L40s  | 26.02 | CUDA 12 |
| **Large Instance** | >95 GB | 2x NVIDIA RTX Pro 6000 | 26.02 | CUDA 13 |

* We recommend using NVIDIA GPU L40s for the best user experience and performance-to-cost ratio for this blueprint, unless otherwise stated in the tutorial. The Advanced or MultiGPU notebooks require one or more 80GB GPUs. We suggest using an 2x RTX Pro 6000 instance.
* Other supported instances, if available in your region:  
* H100  
* A100  
* A10  
* L4  
* GH200  
* 24 GB VRAM or more recommended

* Environment packages can be found on [GitHub](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint/tree/main)

## License

**GOVERNING TERMS**: The [Single Cell Analysis Blueprint scripts](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint), including notebooks and the [RAPIDS AI container](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/rapidsai/containers/notebooks?version=26.02-cuda13-py3.12) are governed by the [Apache 2.0 license](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint?tab=Apache-2.0-1-ov-file#readme), and enables use of separate open source and proprietary software governed by their respective licenses: 
* The [RAPIDS-single cell software](https://github.com/scverse/rapids-singlecell) is governed by the [MIT License](https://github.com/scverse/rapids-singlecell?tab=MIT-1-ov-file#readme). 
* The datasets are governed by CC-BY 4.0. 
* The other supporting open source software components [here](https://github.com/NVIDIA-AI-Blueprints/single-cell-analysis-blueprint/blob/main/requirements.txt) are governed by their accompanying licenses.

## Ethical Considerations

NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and addresses unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy subcards. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).
