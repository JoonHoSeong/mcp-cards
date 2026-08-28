---
name: nemo-mbridge-perf-cpu-offloading
description: Validate and use CPU offloading in Megatron Bridge, including layer-level activation offloading and fractional optimizer state offloading with HybridDeviceOptimizer.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: nemo-mbridge-perf
---

# CPU Offloading

## What It Is
Two independent mechanisms to move data from GPU to CPU memory:

| Mechanism | Config namespace | What gets offloaded | PP restriction |
|---|---|---|---|
| Activation offloading | `model.cpu_offloading*` | Activations/Weights per layer | **PP must be 1** |
| Optimizer offloading | `optimizer.optimizer_cpu_offload` | Adam states via `HybridDeviceOptimizer` | None |

## Quick Decision
| Situation | Recommendation |
|---|---|
| Large MoE (30B+), PP > 1 | **Optimizer offloading** (Activation offload blocked by PP=1) |
| Small/Medium, PP=1, Activation dominated | **Activation offloading** |
| Tunable memory-speed tradeoff | Optimizer offloading with `optimizer_offload_fraction` |
| Throughput priority | **Disable all offloading** |
| CUDA graphs needed | Only Optimizer offloading (Activation offload incompatible) |

## Enablement

### Optimizer CPU offloading (Recommended for Large Models)
```python
cfg.optimizer.optimizer_cpu_offload = True
cfg.optimizer.optimizer_offload_fraction = 1.0
cfg.optimizer.overlap_cpu_optimizer_d2h_h2d = True
```

### Activation CPU offloading (Small/Medium Models Only)
```python
cfg.model.cpu_offloading = True
cfg.model.cpu_offloading_num_layers = 16
cfg.model.cpu_offloading_activations = True
cfg.model.cpu_offloading_weights = False
cfg.model.pipeline_model_parallel_size = 1
cfg.model.recompute_granularity = None
cfg.model.cuda_graph_impl = "none"
```

## Compatibility and Constraints
- **Activation Offload**: Must have `PP=1`, `recompute_granularity=None`, and `cuda_graph_impl="none"`.
- **Optimizer Offload**: Requires `use_distributed_optimizer=True`. No PP or CUDA graph restrictions.
- **MoE Warning**: Activation offloading is practically impossible for large MoE models (e.g., Qwen3-30B) because PP=1 forces all layers on one GPU, exceeding 80GB.

## Failure Diagnosis
| Symptom | Cause | Fix |
|---|---|---|
| `Currently there is no support for PP with CPU offloading` | Activation offload + PP > 1 | Set PP=1 or use Optimizer offloading |
| `CPU offloading does not work when activation recompute is enabled` | Activation offload + recompute | Set `recompute_granularity=null` |
| `CUDA graphs not supported with CPU offloading` | CUDA graphs + activation offload | Set `cuda_graph_impl="none"` |
| Extreme slowdown (>4x) | 100% Optimizer offload CPU bottleneck | Reduce `optimizer_offload_fraction` |

## Verification
```bash
uv run python -m pytest tests/unit_tests/models/test_gpt_full_te_layer_autocast_spec.py -k "cpu_offload" -q
```
Success: Config validation passes; training completes without OOM; memory drops proportionally to offload fraction.