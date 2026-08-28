---
name: nemo-mbridge-perf-moe-hardware-configs
description: MoE training playbooks and throughput bands for H100, B200, GB200, and GB300 platforms.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE Hardware Configuration Reference

Representative playbooks for MoE training across different NVIDIA hardware generations.

## 1. Platform Playbooks

| Platform | Typical Strategy | Primary Tuning Focus |
|---|---|---|
| H100 | DeepEP + Strong PP | Comm overlap and PP efficiency |
| B200 | DeepEP + MXFP8 | Container quality and tuned comm settings |
| GB200 | HybridEP + CPU Cleanup | Host overhead and topology-aware dispatch |
| GB300 | HybridEP + New FP8 Stack | Same as GB200, but with higher performance ceiling |

## 2. Canonical Configuration Rows

| Workload | Hardware | Dispatcher | Layout |
|---|---|---|---|
| DSV3 | H100 | DeepEP | TP=2, EP=64, PP=8, VPP=4 |
| DSV3 | GB200/GB300 | HybridEP | TP=1, EP=64, PP=4, VPP=4 |
| Qwen3 235B | H100 | DeepEP | TP=2, EP=32, PP=8, VPP=4 |
| Qwen3 235B | GB200 | HybridEP | TP=1 or 2, EP=32-64, PP=4, VPP=unspecified |

## 3. Rounded Performance Bands (Planning Ranges)

| Workload Family | Hardware | Typical Band | Representative Shape |
|---|---|---|---|
| DSV3 (Large) | H100 | Low-Mid 100s TFLOPS/GPU | TP2, EP64, PP8, DeepEP |
| DSV3 (Large) | B200 | High 100s TFLOPS/GPU | TP1, EP32, PP8, DeepEP |
| DSV3 (Large) | GB200 | ~1K TFLOPS/GPU | TP1, EP64, PP4, HybridEP |
| Qwen3 235B | H100 | Low 300s TFLOPS/GPU | TP2, EP32, PP8, DeepEP |
| Qwen3 235B | GB200 | High 100s TFLOPS/GPU | TP1 or 2, EP32-64, PP4, HybridEP |

## 4. Common Environment Overrides
For high-performance MoE runs on GB200/GB300:
```bash
export CUDA_DEVICE_MAX_CONNECTIONS=32 # When combining EP overlap + CUDA graphs
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
export NCCL_GRAPH_REGISTER=0
```

## 5. Critical Pitfalls
- **Cargo-Culting**: Do not copy a tracker row exactly; success depends on the specific container, routing mode, and PP layout.
- **VPP Balance**: A poorly chosen VPP split can negate all gains from a superior dispatcher.
- **MFU vs TFLOPS**: When switching precision (BF16 $\leftrightarrow$ FP8), compare absolute TFLOPS rather than MFU, as MFU can be misleading.
