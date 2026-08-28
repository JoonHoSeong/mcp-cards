---
name: nemo-mbridge-perf-parallelism-strategies
description: Guide for choosing and sizing TP/PP/DP/CP/EP parallelism in Megatron Bridge based on model size and hardware topology.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Parallelism Strategy Selection

## 1. Sizing by Model Size (Starting Heuristics)

### Dense Models
| Model size | GPUs | Recommendation |
|---|---|---|
| < 1B | 1-8 | DP only |
| 1-10B | 8-16 | TP=2-4 + DP |
| 10-70B | 16-64 | TP=4-8 + PP=2-4 + DP |
| 70-175B | 64-256 | TP=8 + PP=4-8 + DP |
| 175-500B | 256-1024 | TP=8 + PP=8-16 + CP=2 + DP |

### MoE Models (Sized by **Active** Params)
| Model (total / active) | TP | PP | EP | Notes |
|---|---|---|---|---|
| OLMoE 7B / 1B | 1 | 1 | 8 | Fits single node |
| Moonlight 16B / 3B | 2 | 1 | 8 | Small TP for shared layers |
| DeepSeek-V2 236B / 21B | 1 | 4 | 32 | No TP |
| Qwen3 30B-A3B | 4 | 2 | 4 | |
| DeepSeek-V3 671B / 37B | 2 | 16 | 64 | TP=2 (not 8) |

## 2. Hardware Topology Mapping
- **Single Node (NVLink)**: Maximize TP (up to 8).
- **Multi-Node (InfiniBand)**: Use TP within node, PP/DP across nodes.
- **Limited Network (Ethernet)**: Higher PP, lower TP to minimize cross-node traffic.
- **Rule**: Never run TP across nodes (severe performance loss).

## 3. Sequence Length & CP
| Seq Length | Recommendation |
|---|---|
| < 2K | Standard TP + PP + DP |
| 2K-8K | Enable Sequence Parallel (`sequence_parallel=True`) |
| 8K-32K | Add Context Parallel (CP=2) |
| 32K+ | CP=4-8, consider `a2a+p2p` for large CP |

## 4. The "True" Minimum GPU Count (MoE)
Unlike dense models, MoE meshes (TP*CP and EP*ETP) overlap within each PP stage.
**Correct Formula**: `min_gpus = PP * max(TP * CP, EP * ETP)`
*Do not use the product `PP * TP * CP * EP * ETP` as it over-allocates GPUs.*

## Pitfalls
1. **TP across nodes**: Catastrophic throughput loss.
2. **SP without TP**: `sequence_parallel=True` requires `tensor_model_parallel_size > 1`.
3. **CP Divisibility**: Requires `seq_length % (2 * context_parallel_size) == 0`.
4. **EP on Dense**: `expert_model_parallel_size` is a no-op for non-MoE models.

## Verification
Run a minimal recipe with overridden parallelism:
```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 uv run python -m torch.distributed.run --nproc_per_node=4 \
  scripts/training/run_recipe.py \
  --recipe llama32_1b_pretrain_config \
  model.tensor_model_parallel_size=2 \
  model.pipeline_model_parallel_size=2 \
  model.sequence_parallel=True \
  train.train_iters=3 train.global_batch_size=8 train.micro_batch_size=1 \
  scheduler.lr_warmup_iters=0 validation.eval_iters=0 validation.eval_interval=0 \
  checkpoint.save_interval=0 logger.log_interval=1
```
Success: Exit code 0, finite loss, log shows TP=2 PP=2 DP=1.
