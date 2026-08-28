---
name: nemo-mbridge-perf-megatron-fsdp
description: Enable and configure Megatron FSDP for memory-efficient data parallelism in Megatron-Bridge.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Megatron FSDP

Megatron FSDP provides an alternative to standard DDP by sharding parameters, gradients, and optimizer states across data-parallel ranks.

## 1. Enablement
Essential Bridge overrides for FSDP:
```python
cfg.dist.use_megatron_fsdp = True
cfg.ddp.use_megatron_fsdp = True
cfg.ddp.data_parallel_sharding_strategy = "optim_grads_params"
cfg.ddp.average_in_collective = False
cfg.checkpoint.ckpt_format = "fsdp_dtensor"
```

## 2. Critical Constraints & Pitfalls
- **Checkpoint Format**: FSDP **only** supports `fsdp_dtensor`. Using `torch_dist` will trigger a configuration assertion.
- **Mutual Exclusion**: `use_megatron_fsdp` and `use_torch_fsdp2` are mutually exclusive.
- **TP/PP Mapping**: `use_tp_pp_dp_mapping` is not supported with Megatron FSDP.
- **CPU Offloading**: Only valid when `pipeline_model_parallel_size == 1` and activation recompute is disabled.

## 3. Verification
Run the functional smoke test for FSDP pretraining:
```bash
CUDA_VISIBLE_DEVICES=0,1 uv run python -m torch.distributed.run --nproc_per_node=2 \
  -m pytest tests/functional_tests/training/test_megatron_fsdp.py::TestMegatronFSDP::test_fsdp_pretrain_basic -v -s
```
**Success Criteria**: Pytest reports `1 passed`, finite loss is observed, and no checkpoint format assertions occur.
