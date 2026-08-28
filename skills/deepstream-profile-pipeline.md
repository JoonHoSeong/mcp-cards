---
name: deepstream-profile-pipeline
description: Profile DeepStream pipelines with Nsight Systems to derive optimal configs based on HW ceilings and inference plateaus.
owner: NVIDIA CORPORATION
service: deepstream
version: 0.1.0
reviewed: 2026-04-24
license: CC-BY-4.0 AND Apache-2.0
metadata:
  author: "NVIDIA CORPORATION"
  tags: ["deepstream", "profiling", "nsight-systems", "nvtx", "benchmark"]
---

# DeepStream Profiling Skill

Profile-driven pipeline optimization. This skill replaces guesswork with measured data—specifically the **Inference Plateau Batch** and **HW Ceiling**—to derive every other configuration parameter.

## 🎯 Trigger Intent
Activate this skill when the user requests an **efficient, fast, performant, or optimized** pipeline, or explicitly asks to **benchmark/profile/tune** a pipeline.

## 🚀 The 6-Stage Optimization Flow

### Stage 0 — Preset-Apply (Pre-Generation)
Before generating the pipeline, apply these high-performance defaults:
- **Precision**: `INT8` (if calib file exists), else `FP16`. Never `FP32`.
- **Muxer**: `nvbuf-memory-type = 0` (NVMM), `batched-push-timeout = 1e6 / source_fps`.
- **Decoder**: `num-extra-surfaces = min(batch, 5)`, `cudadec-memtype = 0`.
- **Tracker**: Use `config_tracker_NvDCF_max_perf.yml`.

### Stage 1 — NVTX Coverage Check
Classify pipeline elements as **COVERED** (emits NVTX) or **UNINSTRUMENTED** (e.g., plain GStreamer-core elements). 
- Covered plugins provide precise timing.
- Uninstrumented plugins are handled via closed-form rules.

### Stage 2 — HW Discovery
Use `nvidia-smi` to derive theoretical ceilings:
- **Decode Ceiling**: Based on NVDEC count and source resolution.
- **Compute Ceiling**: SM count $\times$ clock $\times$ ops-per-clock.
- **Memory/PCIe Ceiling**: DRAM bandwidth and PCIe Gen/Width.

### Stage 3 — Inference Micro-Benchmark
Isolate the model by running a `source → streammux → nvinfer → fakesink` pipeline.
- **Sweep**: Batch sizes $\{1, 2, 4, 8, 16, 32\}$.
- **Plateau Batch**: The smallest $B$ where $2 \times B$ yields $< 5\%$ FPS gain.

### Stage 4 — Derive Final Configs
Apply closed-form rules to set all knobs based on `(plateau_batch, HW_ceilings, N_streams, source_res, source_fps)`.
- **Streammux Batch**: Set to `final_batch`.
- **Queues**: `max-size-buffers = final_batch \times 2`.
- **Inference**: Set `batch-size = final_batch`.

### Stage 5 — E2E Profile & Report
Capture a full run using `nsys profile` and extract data via `nsys stats`:
- **Per-Plugin Time**: From `nvtx_sum` (Share of wall time).
- **GPU Kernels**: From `cuda_gpu_kern_sum`.
- **Memcpy**: From `cuda_gpu_mem_time_sum`.

## 📊 Final Report Format (Markdown)
The report must contain:
1. **Hardware & Ceilings**: GPU name, memory, and theoretical ceilings.
2. **Inference Plateau**: Batch size and aggregate peak FPS.
3. **E2E Measured**: Actual FPS and efficiency percentage.
4. **Per-Plugin Table**: Plugin name $\rightarrow$ Wall time % $\rightarrow$ GPU/CPU.
5. **Applied Configs**: List of all derived knobs (e.g., `nvstreammux.batch-size`).

## 🛠️ Technical Constraints
- **Terminal Only**: Use `nsys profile` and `nsys stats`. Do not depend on Nsight Lens or any GUI.
- **No Auto-Injection**: NVTX auto-injection for uninstrumented plugins is out of scope.

## 🔀 Routing & Handoff
- **Pipeline Generation**: Handoff to `deepstream-generate-pipeline` for the initial build.
- **Model Import**: Handoff to `deepstream-import-vision-model` for new model integration.
