---
name: nemo-mbridge-perf-hierarchical-context-parallel
description: Enable and configure hierarchical context parallelism (a2a+p2p) to scale CP beyond KV heads in Megatron-Bridge.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Hierarchical Context Parallel (HCP)

HCP uses nested context-parallel process groups to optimize communication when scaling CP beyond the number of KV heads.

## 1. Enablement
Minimal Bridge override for HCP:
```python
cfg.model.context_parallel_size = 4
cfg.model.cp_comm_type = "a2a+p2p"
cfg.model.hierarchical_context_parallel_sizes = [2, 2]
cfg.dist.use_decentralized_pg = False
```

## 2. Critical Constraints
- **Product Match**: `prod(hierarchical_context_parallel_sizes)` must exactly equal `context_parallel_size`.
- **Divisibility**: `seq_length % (2 * context_parallel_size) == 0`.
- **TE Version**: Requires **Transformer Engine $\ge$ 1.12.0**.
- **PG Implementation**: Currently **MPU-only**. If `use_decentralized_pg=True`, HCP is not activated.

## 3. Pitfalls
- **Silent Failures**: In older stacks, missing `hierarchical_context_parallel_sizes` would silently disable CP communication, leading to high throughput but broken gradients.
- **Configuration Errors**: A mismatch between the product of the list and the total CP size triggers an immediate assertion.

## Verification
Launch a small recipe and inspect the logs for process group initialization:
```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 uv run python -m torch.distributed.run --nproc_per_node=4 \
  scripts/training/run_recipe.py \
  --recipe llama32_1b_pretrain_config \
  model.context_parallel_size=4 \
  model.cp_comm_type=a2a+p2p \
  "model.hierarchical_context_parallel_sizes=[2,2]" \
  train.train_iters=2
```
**Success Criteria**: Logs explicitly show `HIERARCHICAL_CONTEXT_PARALLEL_GROUPS` being created. If only `CONTEXT_PARALLEL_GROUP` is seen, HCP is inactive.
