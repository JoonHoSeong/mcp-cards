---
name: quantitative-portfolio-optimization
title: "Quantitative Portfolio Optimization"
description: "Enable fast, scalable, and real-time portfolio optimization for financial institutions."
publisher: "nvidia"
type: blueprint
updated: "2026-02-17T19:30:46.422Z"
canonical: https://build.nvidia.com/nvidia/quantitative-portfolio-optimization
source_url: https://build.nvidia.com/qc69jvmznzxy/quantitative-portfolio-optimization.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# Quantitative Portfolio Optimization

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `quantitative-portfolio-optimization` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/quantitative-portfolio-optimization](https://build.nvidia.com/nvidia/quantitative-portfolio-optimization) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/quantitative-portfolio-optimization.md](https://build.nvidia.com/qc69jvmznzxy/quantitative-portfolio-optimization.md) |
| Source snapshot | [../raw/quantitative-portfolio-optimization.md](../raw/quantitative-portfolio-optimization.md) |
| Last updated by publisher | 2026-02-17T19:30:46.422Z |

**Summary:** Enable fast, scalable, and real-time portfolio optimization for financial institutions.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

The Quantitative Portfolio Optimization developer example enables GPU-accelerated, real-time, scalable portfolio optimization for financial institutions. Leveraging NVIDIA® cuOpt™ and RAPIDS, this developer example delivers near-real-time solutions for large-scale Mean-CVaR portfolio optimization problems, allowing enterprises to model advanced risk measures and optimize complex portfolios in accelerated time.

## Architecture Diagram
![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/quantitative-portfolio-optimization/diagram.jpg)

## Key Features
* **End-to-End GPU Workflow:** Accelerates the portfolio allocation problem using NVIDIA® cuOpt™, delivering near-real-time optimization.

* **Flexible Financial Model Building:** Fully customizable constraints, including CVaR, leverage, budgets, turnover, and cardinality limits.

* **Data-driven Risk Modeling:** Utilizes CVaR as a risk measure that is scenario-based and requires no assumptions about the asset return distribution.   

* **Full Pipeline Support:** Provides tools for performance evaluation,  strategy backtesting, benchmarking, and visualization.

* **Easy Benchmarking:** Streamlines the process of benchmarking against CPU-based libraries and solvers.

* **Scalable & Efficient:** Excels at solving large LP and MILP problems, leveraging NVIDIA libraries for pre- and post-optimization acceleration.

## Techniques and Architecture
* cuML: Uses cuML KDE for learning the underlying joint return distribution of all assets and samples from the learned distribution  

* RAPIDS/cuDF: (optional) use cuDF for returns calculations and backtesting operations

* cuOpt: Uses LP/MILP solvers to solve problems. Problem sizes depend on the number of scenarios sampled in the scenario generation step.

* CVXPY: Uses CVXPY for flexible optimization model building; using CVXPY makes it very easy for customers to benchmark cuOpt solvers against other solvers. We also support building models in the cuOpt Python API. 

* Advanced Risk Modeling: Supports Conditional Value-at-Risk (CVaR) and scenario-based stress testing to capture tail risk and asymmetric return distributions.

* GPU-Accelerated Optimization: Uses NVIDIA® cuOpt™ to accelerate portfolio optimization, handling tens of thousands of variables and constraints in near-real time.

* Scalable Strategy backtesting: Efficiently solves LP and MILP problems at scale during backtesting, reducing infrastructure overhead and operational latency.

## Model Architecture
**Architecture Type**: x86/aarch64 

**Network Architecture**: Not Applicable (N/A)

**Container Version**: nvcr.io/nvidia/pytorch:25.08-py3 

## Minimum System Requirements
#### Hardware Requirements
* NVIDIA H100 SXM (compute capability >= 9.0) and above
* CPU: 32+ cores
* NVMe SSD Storage: 100+ GB free space

##### OS requirements
- Linux distributions with glibc>=2.28 (released in August 2018):
- Arch Linux (minimum version 2018-08-02)
- Debian (minimum version 10.0)
- Fedora (minimum version 29)
- Linux Mint (minimum version 20)
- Rocky Linux / Alma Linux / RHEL (minimum version 8)

#### Software Requirements
* cuopt-cu13==25.10.0  
* cuml-cu13==25.10.0

#### 3rd Party  Technologies
* numpy
* pandas
* cvxpy
* scipy
* scikit-learn
* seaborn

## Ethical Considerations
NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy Subcards. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## Terms of Use
The Jupyter notebook, developer example scripts, RAPIDS and NVIDIA cuOpt software are governed by the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0.txt)
