---
name: nemo-mbridge-perf-tp-dp-comm-overlap
description: Operational guide for enabling TP, DP, and PP communication overlap in Megatron-Bridge, including config knobs, code anchors, pitfalls, and verification.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# TP / DP / PP Communication Overlap Guide

This guide provides the operational steps to enable and verify communication overlap in Megatron-Bridge, maximizing GPU utilization by hiding collective communication latency behind compute.

## 1. Enablement Strategy

### Minimal Configuration Override
To enable basic overlap, update the configuration as follows:

```python
from megatron.bridge.training.comm_overlap import CommOverlapConfig

# Prerequisites for TP Overlap
cfg.model.tensor_model_parallel_size = 4
cfg.model.sequence_parallel = True 
cfg.model.pipeline_model_parallel_size = 4
cfg.model.virtual_pipeline_model_parallel_size = 2

cfg.comm_overlap = CommOverlapConfig(
    tp_comm_overlap=True,
)

# DP Overlap Settings
cfg.ddp.use_distributed_optimizer = True
cfg.ddp.overlap_grad_reduce = True
cfg.ddp.overlap_param_gather = True
```

### High-Performance TP Presets
For specific hardware (e.g., H100), use pre-defined buffer configurations to avoid manual tuning:

```python
from megatron.bridge.training.comm_overlap import userbuffers_bf16_h100_h12288_tp4_mbs1_seqlen2048

cfg.comm_overlap.tp_comm_overlap_cfg = userbuffers_bf16_h100_h12288_tp4_mbs1_seqlen2048
```

### Precision Control
Overlap performance is tied to mixed-precision settings:
- `cfg.mixed_precision.grad_reduce_in_fp32 = False`
- `cfg.mixed_precision.fp8_param_gather = False`

## 2. Technical Implementation Anchors
The following logic governs how overlap is activated within the Bridge:

- **TP Overlap Gate**: Triggered only if `tp_comm_overlap=True` AND `tensor_model_parallel_size >= 2` AND `sequence_parallel=True` AND Transformer Engine (TE) is available.
- **PP Overlap Selection**: 
    - If `PP > 1` AND `VPP > 1` $\rightarrow$ `overlap_p2p_comm=True` (P2P overlap).
    - If `PP > 1` AND `VPP = 1` $\rightarrow$ `batch_p2p_comm=True` (Batch P2P).
- **DP Overlap Defaults**: Automatically enabled if `data_parallel_size > 1` with a default `bucket_size` of 128MB (parameter count based).

## 3. Launch-Time Environment Tuning
Certain overlap optimizations are controlled via environment variables during the plugin launch:
- `CUDA_DEVICE_MAX_CONNECTIONS`: Essential when combining EP overlap with CUDA graphs.
- `NVTE_FWD_LAYERNORM_SM_MARGIN` / `NVTE_BWD_LAYERNORM_SM_MARGIN`: Tunes the SM margin for LayerNorm to prevent compute/comm collisions.

## 4. Critical Pitfalls
- **Silent Disabling**: TP overlap will silently fail to activate if `sequence_parallel` is `False`.
- **VPP Dependency**: PP overlap is not a global switch; it is automatically selected based on the presence of Virtual Pipeline stages.
- **Bucket Size Misunderstanding**: `bucket_size` refers to the **number of parameters**, not the raw byte size.
- **Mixed Precision Conflict**: Attempting to tune DDP overlap without aligning `grad_reduce_in_fp32` can lead to precision errors or performance regressions.

## 5. Verification Workflow

### Step 1: Unit Test Validation
Run the dedicated communication overlap tests to ensure the logic is sound:
```bash
uv run python -m pytest tests/unit_tests/training/test_comm_overlap.py -q
```
*Success Criteria: 26 passed.*

### Step 2: Plugin Wiring Validation
Verify that the environment variables are correctly wired through the recipe plugins:
```bash
uv run python -m pytest tests/unit_tests/recipes/test_run_plugins.py -q
```
