---
name: nemo-mbridge-perf-memory-tuning
description: GPU memory optimization in Megatron Bridge: expandable segments, PEFT+SP input re-gather, parallelism resizing, and OOM diagnosis.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Memory Tuning

GPU OOM failures during training often stem from memory **fragmentation** rather than raw capacity.  The most effective fix is `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True`.

## Quick Decision Matrix

| Symptom | Cause | First Action | Long-term Fix |
|---|---|---|---|
| OOM on single rank (headroom on others) | Fragmentation | `expandable_segments:True` | Check `PYTORCH_CUDA_ALLOC_CONF` |
| OOM with `expandable_segments` set | Capacity Limit | `nvidia-smi` check | Increase PP / Distributed Optimizer |
| Pre-launch estimate > GPU capacity | Model too large | `estimate_training_memory` | Adjust TP/PP/CP/EP |
| LoRA+SP high activation memory | Gathered inputs retained | `sequence_parallel_input_regather=True` | Verify eligible projections |
| `ValueError: PP + CPU offloading` | Incompatible Config | Check PP size | Set PP=1 or disable CPU offload |

## Enablement

### 1. Memory Fragmentation Fix (Zero Cost)
Set in environment before launch:
```bash
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
```
*Note: Incompatible with `--use-nccl-ub`. For CUDA graphs on pre-Blackwell, set `NCCL_GRAPH_REGISTER=0`.*

### 2. LoRA + SP Input Re-gather (Memory-for-Comm Tradeoff)
Reduces activation memory for eligible column-parallel LoRA-A projections:
```python
cfg.peft = LoRA(
    sequence_parallel_input_regather=True,
)
```

### 3. Parallelism Resizing (Capacity Fix)
| Strategy | Memory Effect | Throughput Cost | Recommendation |
|---|---|---|---|
| Increase PP | Fewer layers/stage | Moderate (~6%) | Use if GPU count allows |
| Increase TP | Fewer params/GPU | Severe (-28% on 70B) | Last resort |
| Dist. Optimizer | Shards optimizer state | Minimal (~1%) | Recommended for large models |

## Critical Constraints & Pitfalls
- **CPU Offloading**: strictly **blocked when PP > 1**.
- **VPP (Virtual Pipeline Parallelism)**: A throughput optimization. It does NOT meaningfully reduce peak memory. Do not use it as a memory fix.
- **TP Across Nodes**: Destroys throughput. Keep TP within a single NVLink domain.

## Verification
Check if expandable segments are active:
```python
import os
assert "expandable_segments:True" in os.environ.get("PYTORCH_CUDA_ALLOC_CONF", "")
```
Verify LoRA SP re-gather with MCore backward-parity test:
```bash
uv run python -m torch.distributed.run --nproc_per_node=2 -m pytest \
  tests/unit_tests/peft/test_lora_sp_input_regather_distributed.py
```
