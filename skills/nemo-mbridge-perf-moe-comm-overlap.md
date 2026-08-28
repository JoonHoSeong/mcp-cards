---
name: nemo-mbridge-perf-moe-comm-overlap
description: Optimize MoE expert-parallel communication overlap (alltoall, flex/DeepEP) to hide dispatch/combine latency.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE Communication Overlap

MoE communication overlap hides the cost of token dispatch/combine all-to-all communication by running it concurrently with expert FFN compute.

## 1. Enablement
Essential Bridge overrides:
```python
cfg.comm_overlap.overlap_moe_expert_parallel_comm = True
cfg.comm_overlap.delay_wgrad_compute = True # Optional: further overlap wgrad
cfg.model.moe_shared_expert_overlap = False # Mandatory: must be disabled
```

### Prerequisites
- **Parallelism**: `expert_model_parallel_size > 1` and `num_moe_experts > 1`.
- **Precision**: BF16 or FP16.
- **Pipeline**: If $PP > 1$, `virtual_pipeline_model_parallel_size` must be non-None.
- **Dispatcher**: `moe_token_dispatcher_type` must be `"alltoall"` or `"flex"`.

## 2. Recompute & Graph Interaction
- **Incompatible**: Full recompute (`recompute_granularity = "full"`) is not a suitable companion for overlap.
- **Recommendation**: Use **selective recompute** when enabling communication overlap.
- **Constraints**: `delay_wgrad_compute` adds additional constraints if CUDA-graph scopes include attention or MoE-router work.

## 3. Pitfalls
- **Shared Expert Clash**: `moe_shared_expert_overlap` and `overlap_moe_expert_parallel_comm` are mutually exclusive.
- **The VPP Wall**: MoE overlap requires VPP when pipeline parallelism is active; without it, scheduling cannot interleave.
- **Flex Dispatcher Gate**: `moe_flex_dispatcher_backend="deepep"` does nothing unless `moe_token_dispatcher_type = "flex"`.

## Verification
Inspect logs during initialization for `CommOverlapConfig` validation. A clean startup confirms prerequisites are met.
Run a performance smoke test, varying only one knob at a time:
```bash
uv run python scripts/performance/run_script.py \
  -m qwen -mr qwen3_30b_a3b --task pretrain -g h100 -c bf16 -ng 16 -gn 8 --max_steps 8 \
  --cuda_graph_impl none --moe_flex_dispatcher_backend None --moe_a2a_overlap false \
  comm_overlap.overlap_moe_expert_parallel_comm=true \
  comm_overlap.delay_wgrad_compute=false \
  model.moe_shared_expert_overlap=false
```
**Success Criteria**: Training runs without hangs/assertions, and throughput improves for the target MoE shape.
