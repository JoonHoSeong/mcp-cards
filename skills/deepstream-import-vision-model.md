---
name: deepstream-import-vision-model
description: Bring vision models from HF/NGC into DeepStream with automation: ONNX download, TRT build, custom parser, and benchmark reporting.
license: CC-BY-4.0 AND Apache-2.0
metadata:
  author: "NVIDIA CORPORATION"
  version: "1.2.2"
  tags: ["deepstream", "tensorrt", "huggingface", "ngc", "onnx", "benchmark"]
---

# DeepStream Import Vision Model

End-to-end automation for importing object detection models from HuggingFace or NVIDIA NGC into a DeepStream pipeline. This skill manages the entire lifecycle from model acquisition to high-fidelity benchmark PDF reports.

## 🎯 Scope & Constraints
- **Supported Models**: Object detection only.
- **Architectures**: Fail fast on classification, segmentation, or other non-detection architectures.
- **Requirement**: Reference documents MUST be read before starting each phase.

## 🚀 The 8-Step Automation Pipeline

### Phase 1: Model Acquire (Steps 1–3)
*Reference: `references/model-acquire.md`*
1. **Browse & Detect**: Identify model on HF/NGC, detect format.
2. **Download/Export**: Download ONNX or export SafeTensors $\rightarrow$ ONNX via `optimum-cli`.
3. **Label Extraction**: Extract class labels and map them to `labels.txt`.

### Phase 2: Engine Build (Steps 4–5)
*Reference: `references/engine-build.md`*
4. **Dynamic TRT Build**: Build a dynamic TensorRT engine using `trtexec`.
5. **Compute-Peak Bench**: Run `trtexec` at BS=1 and BS=MAX_BS to derive `PEAK_GPU_STREAMS`.

### Phase 3: DS Pipeline Execution (Steps 6–7)
*Reference: `references/pipeline-run.md`*
6. **Custom Parser**: Build and link a custom C++ bbox parser (`NvDsInferObjectDetectionInfo`).
7. **Multi-Stream Benchmark**: Run benchmarks across varying stream counts; generate KITTI detection dumps for validation.

### Phase 4: Report Generation (Step 8)
*Reference: `references/report-generation.md`*
8. **Final Report**: Generate 5 charts $\rightarrow$ HTML $\rightarrow$ PDF via `md-to-html-pdf.py`.

## ⚠️ Critical Operational Rules

### 1. Bounding Box Parser Zero-Init (MANDATORY)
Always initialize the parser struct to zero:
`NvDsInferObjectDetectionInfo obj = {};`
**Failure to do this** leaves `rotation_angle` uninitialized, resulting in tilted/diagonal bounding boxes.

### 2. Encoder Fallback Chain
When NVENC is unavailable, follow this strict priority:
`nvv4l2h264enc` $\rightarrow$ `theoraenc + oggmux` (Output: `.ogv`) $\rightarrow$ `skipped`.
**Prohibited**: `x264enc` and `openh264enc`.

### 3. Engine Naming & Batching
- **Naming**: Always use `{model}_dynamic_b{MAX_BS}.engine`.
- **Consistency**: In DS runs, `batch-size` must always equal the number of streams.

### 4. Video Source Default
Default is always `sample_720p.mp4`. Only use alternatives if provided via `DS_VIDEO`.

## 📂 Mandatory Directory Structure
All model artifacts must be organized as follows:
```text
models/{model_name}/
  model/           <- ONNX files
  parser/          <- .cpp, Makefile, .so
  config/          <- nvinfer config, labels.txt
  benchmarks/
    engines/       <- .engine, timing.cache, logs
    b1/            <- BS=1 logs
    b{MAX_BS}/     <- BS=MAX_BS logs
    ds/            <- DS benchmark logs
  reports/         <- .md, .html, .pdf, charts/
  samples/         <- .mp4 / .ogv, kitti_output/
```

## 🛠️ Tooling & Scripts
- **Shared Venv**: Use `build/.venv_optimum` for all models.
- **Report Engine**: Always use `scripts/report/md-to-html-pdf.py`.
- **Validation Gate**: Do NOT proceed to Step 7 if KITTI frame count is zero or detection rate < 90%.

## 🔀 Routing & Handoff
- **Profiling**: For performance tuning of the imported model, hand off to `deepstream-profile-pipeline`.
- **Pipeline Generation**: For using the model in a general pipeline, hand off to `deepstream-generate-pipeline`.
