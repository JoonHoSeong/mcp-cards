---
name: nemo-mbridge-perf-sequence-packing
description: Validate and use packed sequences for LLM/VLM training in Megatron-Bridge, managing CP constraints and THD layout.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Sequence Packing

## 1. Enablement

### Offline Packed SFT (LLM Finetuning)
Used to eliminate padding waste. Requires `micro_batch_size = 1`.
```python
cfg.dataset.enable_offline_packing = True
cfg.dataset.offline_packing_specs = PackedSequenceSpecs(
    packed_sequence_size=4096,
    pad_seq_to_mult=1, # See CP constraints below
)
```

### In-Batch Packing (VLM Finetuning)
Packs multiple samples into a batch during collation. Requires `micro_batch_size > 1`.
```python
cfg.dataset.enable_in_batch_packing = True
```

### Long-Context Baseline
```python
cfg.model.seq_length = 16384
cfg.model.context_parallel_size = 2
```

## 2. Critical Constraints

### Context Parallel (CP) Alignment
When CP is enabled, the packed sequence length must be divisible by `2 * context_parallel_size`.
**For Offline Packing**: Explicitly set `pad_seq_to_mult = 2 * CP` (or `lcm(2*CP, CP*TP)` if SP is also enabled).

### CUDA Graph Requirements
If using CUDA graphs with packed paths:
- Set `cfg.dataset.offline_packing_specs.pad_cu_seqlens = True`.
- Set `cfg.dataset.dataset_kwargs["pad_to_max_length"] = True`.
- **Mandatory**: A metadata JSON file must exist alongside the packed dataset.

## 3. Pitfalls
1. **Mutual Exclusion**: `enable_offline_packing` and `enable_in_batch_packing` cannot both be True.
2. **Micro-batch Rule**: Offline packing $\rightarrow$ MBS=1; In-batch packing $\rightarrow$ MBS > 1.
3. **Loss Masking**: Synthetic padding rows must retain an all-zero loss mask to avoid gradient corruption.
4. **MTP Incompatibility**: Multi-Token Prediction (MTP) finetuning is currently incompatible with packed sequences.

## Verification
Run focused unit tests for packing logic and CP divisibility:
```bash
uv run python -m pytest tests/unit_tests/training/utils/test_packed_seq_utils.py -v && \
uv run python -m pytest tests/unit_tests/training/test_config.py -k "packed_sequence or enable_in_batch_packing" -v && \
uv run python -m pytest tests/unit_tests/data/packing/test_in_batch.py -v
```
Success: All tests pass, configuration validation correctly rejects mutually exclusive settings.
