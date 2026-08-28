---
name: nemo-mbridge-perf-expert-parallel-overlap
description: Optimize MoE expert-parallel communication overlap (alltoall, flex/DeepEP) to hide dispatch/combine latency.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE Expert-Parallel Overlap

EP overlap hides the cost of token dispatch/combine all-to-all communication by running it concurrently with expert FFN compute.

## 1. Dispatcher Decision Matrix

| Dispatcher | Backend | Recommendation |
|---|---|---|
| `alltoall` | Standard MoE all-to-all | First rollout, broadest compatibility |
| `flex` | DeepEP / HybridEP | High-performance overlap on Ampere/Hopper/Blackwell |

## 2. Enablement

### alltoall Dispatcher (Correctness-First)
Use this as a baseline before moving to flex dispatch:
```python
cfg.comm_overlap.overlap_moe_expert_parallel_comm = True
cfg.comm_overlap.delay_wgrad_compute = False
cfg.model.moe_shared_expert_overlap = False
cfg.model.moe_token_dispatcher_type = "alltoall"
```

### flex Dispatcher (High Performance)
Requires calling the activation helper:
```python
from megatron.bridge.training.flex_dispatcher_backend import apply_flex_dispatcher_backend

cfg.comm_overlap.overlap_moe_expert_parallel_comm = True
cfg.comm_overlap.delay_wgrad_compute = True
apply_flex_dispatcher_backend(cfg.model, moe_flex_dispatcher_backend="deepep") # or "hybridep"
```

## 3. Critical Constraints & Pitfalls
- **EP Required**: `expert_model_parallel_size > 1` and `num_moe_experts > 1`.
- **Mutual Exclusion**: MoE overlap is **incompatible with `moe_shared_expert_overlap`**.
- **Recompute Clash**: Incompatible with `recompute_granularity = "full"`.
- **Hardware Gating**: DeepEP/HybridEP only support Ampere, Hopper, B200, B300.
- **Version Requirement**: PyTorch $\ge$ 2.6.0.
- **VPP Requirement**: If $PP > 1$, `virtual_pipeline_model_parallel_size` must be set.

## 4. Failure Diagnosis

| Symptom | Cause | Fix |
|---|---|---|
| `assert expert_model_parallel_size > 1` | EP not configured | Set EP > 1 |
| `assert moe_token_dispatcher_type` | Wrong dispatcher | Use `"alltoall"` or `"flex"` |
| Hang during training | PyTorch < 2.6.0 | Upgrade PyTorch |
| No gain from flex dispatcher | Helper not called | Call `apply_flex_dispatcher_backend(...)` |

## Verification
Run the MoE communication overlap unit tests:
```bash
uv run python -m pytest \
  tests/unit_tests/training/test_comm_overlap.py -k "moe" \
  tests/unit_tests/training/test_deepep.py -q
```
Success: Tests pass, logs show `overlap_moe_expert_parallel_comm = True`, and throughput improves without affecting loss convergence.
