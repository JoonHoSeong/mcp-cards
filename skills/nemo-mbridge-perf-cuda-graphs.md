---
name: nemo-mbridge-perf-cuda-graphs
description: Validate and use CUDA graph capture in Megatron Bridge, including local full-iteration graphs and Transformer Engine scoped graphs.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: nemo-mbridge-perf
---

# CUDA Graphs

## What It Is
CUDA graphs capture GPU operations once and replay them with minimal host-driver overhead.

| `cuda_graph_impl` | Mechanism | Scope support |
|---|---|---|
| `"local"` | `FullCudaGraphWrapper` wrapping fwd+bwd | `full_iteration` |
| `"transformer_engine"` | `make_graphed_callables()` per layer | `attn`, `mlp`, `moe`, `moe_router`, etc. |

## Quick Decision
- **Default**: Use `transformer_engine` scoped graphs.
- **Dense**: `attn` $\rightarrow$ `mlp`.
- **MoE (Dropless)**: `attn`, `moe_router`, `moe_preprocess`.
- **Local Full**: Use `local` + `full_iteration` only for full-iteration capture requirements.
- **Recompute**: TE-scoped graphs pair with **selective recompute**. Full recompute usually requires `local` or disabling graphs.

## Enablement

### Local full-iteration graph
```python
cfg.model.cuda_graph_impl = "local"
cfg.model.cuda_graph_scope = ["full_iteration"]
cfg.model.cuda_graph_warmup_steps = 3
cfg.model.use_te_rng_tracker = True
cfg.rng.te_rng_tracker = True
cfg.rerun_state_machine.check_for_nan_in_loss = False
```

### TE scoped graph (Dense/MoE)
```python
cfg.model.cuda_graph_impl = "transformer_engine"
cfg.model.cuda_graph_scope = ["attn", "moe_router", "moe_preprocess"] # adjust for dense/moe
cfg.model.cuda_graph_warmup_steps = 3
cfg.model.use_te_rng_tracker = True
cfg.rng.te_rng_tracker = True
```

## Compatibility and Constraints
- **Mandatory**: `use_te_rng_tracker = True` and `rng.te_rng_tracker = True`.
- **NaN Check**: `full_iteration` scope requires `check_for_nan_in_loss = False`.
- **Static Shapes**: Fixed `seq_length` and `micro_batch_size` are required.
- **MoE**: `moe` scope and `moe_router` scope are mutually exclusive.
- **GPU Arch**: For compute capability < 10.0, set `NCCL_GRAPH_REGISTER=0` with `expandable_segments:True`.
- **Conflict**: Incompatible with **CPU offloading**.

## Pitfalls
- **Memory Overhead**: Graphs pin buffers; `full_iteration` can increase peak memory by 1.5-2x.
- **Variable SeqLen**: Any change in sequence length breaks the graph.
- **Full Recompute Conflict**: `recompute_granularity="full"` asserts with TE-scoped graphs. Use `selective` recompute instead.

## Verification
```bash
uv run python -m pytest tests/unit_tests/training/test_config.py -k "cuda_graph" -q
```
Success: Unit tests pass; functional smoke test completes training without NCCL errors or illegal memory access.