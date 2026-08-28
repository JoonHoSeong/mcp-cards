---
name: nemo-mbridge-perf-activation-recompute
description: Validate and use selective and full activation recompute in Megatron Bridge to reduce GPU memory usage at the cost of extra compute.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: nemo-mbridge-perf
---

# Activation Recompute

## What It Is
Activation recompute trades GPU compute for memory by discarding intermediate activations during the forward pass and recomputing them during backward. Megatron Bridge supports two granularities:

| Granularity | What you specify | What gets recomputed | Memory savings | Compute cost |
|---|---|---|---|---|
| `selective` | `recompute_modules` list | specific submodules within each layer | moderate | low to high |
| `full` | `recompute_num_layers` + `recompute_method` | entire transformer layers | strongest | highest |

## Quick Decision
1. **Fragmentation First**: Rule out allocator fragmentation using `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True`.
2. **Selective First**: Start with `recompute_granularity="selective"` and `recompute_modules=["core_attn"]`.
3. **Incremental Add**: Add modules by cost: `layernorm` (cheap/low save) $\rightarrow$ `mlp` (expensive/high save).
4. **Full as Last Resort**: Use `recompute_granularity="full"` only when selective fails.
5. **CUDA Graph Conflict**: With FP8 or TE-scoped CUDA graphs, avoid `full` recompute unless `cuda_graph_scope` is `full_iteration`.

## Enablement

### Selective recompute
```python
cfg.model.recompute_granularity = "selective"
cfg.model.recompute_modules = ["core_attn"] 
```

### Full-layer recompute
```python
cfg.model.recompute_granularity = "full"
cfg.model.recompute_method = "uniform"
cfg.model.recompute_num_layers = 4
```

## Compatibility and Constraints
- **TE-Scoped Graph Conflict**: `recompute_granularity="full"` is **incompatible** with TE-scoped graphs (`attn`, `mlp`, etc.). Use `selective` or set `cuda_graph_impl="none"`.
- **PP Constraint**: CPU offloading (`cpu_offloading=True`) is **incompatible** with PP > 1.
- **Sequence Parallel**: `distribute_saved_activations=True` cannot be combined with `sequence_parallel=True`.

## Failure Diagnosis
| Symptom | Cause | Fix |
|---|---|---|
| >15% GPU utilization drop | `mlp` recompute on large FFN | Remove `mlp`, lower MBS, or use CPU offload (PP=1) |
| Still OOM after layernorm | Activations too small | Switch to higher-impact module or full-layer recompute |
| `AssertionError: full recompute is only supported with full iteration CUDA graph` | Layer-level recompute + TE-scoped graph | Use `selective` or `cuda_graph_impl="none"` |
| ValueError: PP + CPU offloading | `cpu_offloading=True` + PP > 1 | Disable CPU offloading or set PP=1 |

## Verification
```bash
uv run python -m pytest tests/unit_tests/training/test_config.py -k "recompute" -q
```
Success: Unit tests pass; no assertion errors during config validation.