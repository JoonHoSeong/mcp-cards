---
name: gpu-query-engine
title: "GPU Query Engine (GQE)"
description: "Build a GPU-accelerated SQL engine using NVIDIA's GQE reference architecture and libcudf with NVIDIA's open-source C++ APIs."
publisher: "nvidia"
type: blueprint
updated: "2026-08-03T14:47:59.935Z"
canonical: https://build.nvidia.com/nvidia/gpu-query-engine
source_url: https://build.nvidia.com/qc69jvmznzxy/gpu-query-engine.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# GPU Query Engine (GQE)

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `gpu-query-engine` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/gpu-query-engine](https://build.nvidia.com/nvidia/gpu-query-engine) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/gpu-query-engine.md](https://build.nvidia.com/qc69jvmznzxy/gpu-query-engine.md) |
| Source snapshot | [../raw/gpu-query-engine.md](../raw/gpu-query-engine.md) |
| Last updated by publisher | 2026-08-03T14:47:59.935Z |

**Summary:** Build a GPU-accelerated SQL engine using NVIDIA's GQE reference architecture and libcudf with NVIDIA's open-source C++ APIs.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

Experience speed-of-light analytics query processing with GPU Query Engine (GQE). Designing a GPU-accelerated SQL engine is not trivial, but required to achieve the best performance. As data grows in size and complexity, the conventional way of data processing can become the bottleneck of pipelines crucial to making fast, informed business decisions.

The NVIDIA GQE Blueprint solves this by providing a reference architecture designed, developed and benchmarked by NVIDIA. With a dedicated team of database researchers and CUDA experts, GQE pushes the frontier of high-performance databases with its state-of-the art performance. Built on top of [NVIDIA cuDF](https://github.com/rapidsai/cudf) and [NVCOMP](https://developer.nvidia.com/nvcomp). This blueprint offers guidance to database developers on how to design the best-in-class GPU-accelerated SQL database engine for quantitative research.

## Use Cases

1. **In-CPU-memory Query Processing**

Cache data tables in CPU memory for faster processing. 

* Techniques: Ingest data into host memory and stored as GQE custom data layout. Query execution can process data directly from host memory.  
* Outcome: Faster than reading from disk, allow for partition pruning on compressed data via custom memory layout.

### Architecture Diagram
![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/gpu-query-engine/diagram.jpg)

2. **Parquet-on-disk Query Processing**

Query directly from Parquet files to leverage disk capacity.

* Techniques: Query directly from Parquet files stored on disk.  
* Outcome: Allow processing on large datasets based on disk capacity.  
* Limitation: Slower than in-CPU-memory case, does not currently support partition pruning

## Key Benefits
* Reduced Latency: Lower the execution time of complex analytics query processing via GPU acceleration.
* Reduced Cost and Power Consumption: Lower the overall cost and power consumption owing to reduced latency.
* Flexible Evaluation and Integration: Use Substrait to test your own plans or even a different SQL front-end to observe the benefits for your specific use cases.
* Ease of Development: Built on top of [libcudf](https://docs.rapids.ai/api/libcudf/stable/developer_guide), you can learn from GQE to build your own GPU-accelerated database with minimal to zero CUDA development.
* Stay at the Frontier of Database Innovation: GQE is backed by a team of database researchers and CUDA experts actively researching and integrating new ideas.

## Key Features
* Data Pipeline Acceleration: Complex analytics is known to contribute to latency and cost of many data pipelines; GQE delivers significant speedups via GPU acceleration.
* Multi-Storage Solution: GQE can query directly from Parquet format on disk, or from CPU and/or GPU memory, allowing flexible caching scenarios.
* Multi-GPU Support: Improves performance further by scaling out.
* Native Hardware Decompression Engine Support: Leverage Blackwell’s new feature: HW DE – lowering IO requirements while offloading decompression to dedicated hardware.
* Substrait Compatible: Ability to bring your own plan or framework front-end.  
* Achieve the Best Performance without Trial-and-Error: GQE developers spent years researching the best way to design GPU-acceleration for databases so you don’t have to. Learnings are published in our [blog](https://developer.nvidia.com/blog/designing-gpu-accelerated-query-engines-with-nvidia-gqe/) and [OSS](https://github.com/rapidsai/gqe).

## Minimum System Requirements

**Hardware Requirements**

* To run with HW DE: B100/B200/B300  
* Memory: Values based on input data size (8GB recommended for 100GB datasets)  
* Storage: Varies based on input data size

**OS Requirements**

* Any Docker Engine-capable Linux distro e.g. Ubuntu 24.04 

**Software Dependencies**

* Docker Engine (all other dependencies installed via Dockerfile)

## Software Used in This Blueprint
**NVIDIA Technology**

* cuCollections  
* cuDF  
* nvCOMP  
* NVSHMEM

**3rd Party Software**

* Arrow Flight SQL  
* Boost  
* DataFusion  
* Substrait

## Ethical Considerations

NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI and data processing applications. When downloaded or used in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## License

Use of the reference architecture in this blueprint is governed by the [Apache License 2.0](https://github.com/rapidsai/gqe/blob/main/LICENSE).

## Terms of Use

The use of the GQE source code and its derivative is governed by the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0). The example dataset is a derivative of, and is not the [official TPC-H](https://www.tpc.org/tpch/) dataset. The underlying libraries nvCOMP is governed by [NVIDIA Software License Agreement (EULA)](https://docs.nvidia.com/cuda/nvcomp/license.html) and the [NVIDIA Software Development Kits License Agreement](https://docs.nvidia.com/cuda/nvcomp/license.html), and libcudf is governed by [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
