---
name: nemo-mbridge-perf-moe-long-context
description: Long-context MoE training guidance for Megatron Bridge. Covers CP sizing, selective recompute, dispatcher choices, and practical patterns from DSV3, Qwen3, and Qwen3-Next long-context experiments.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE Long-Context Training Guide

Training MoE models at long sequence lengths (16K–256K tokens) requires a fundamental shift in memory management and parallelism.

## 1. Memory Constraint Shift
Once sequence length exceeds the 4K regime, **attention memory and activation residency** become the dominant constraints, overshadowing simple parameter counts.

### The Long-Context Toolbox
To avoid OOM and maintain throughput, you must combine:
- **Context Parallelism (CP)**: To shard the attention computation.
- **Selective Recompute**: To reduce activation memory without the full cost of re-computation.
- **Lower Precision**: FP8 or MXFP8 to halve memory footprints.
- **CPU Offload**: Specifically for optimizer states.

## 2. CP Sizing & Scaling Rules
Selecting the correct CP size is the most critical decision for long-context stability.

### CP Sizing Heuristics
- **Baseline Guess**: `CP ~= seq_len / 4096`. Round up to the nearest power of two.
- **DP Preservation**: Maximize CP only as much as needed to fit. If CP, EP, TP, and PP together squeeze DP to 1, the training becomes extremely brittle.
- **TP Trade-off (NVL72)**: On GB200/GB300, you can sometimes trade a small amount of CP for TP to gain better inter-node efficiency.

### Scaling Patterns (Representative)
| Model Family | Target SeqLen | Recommended Layout | Precision | Dispatcher |
|---|---|---|---|---|
| DSV3 | 128K | TP=1, CP=32, EP=32, PP=8, VPP=4 | FP8 | DeepEP |
| DSV3 | 256K | TP=1, CP=64, EP=32, PP=8, EDP=2, VPP=4 | FP8 | DeepEP |
| Qwen3 235B | 128K | TP=4, CP=4, EP=32, PP=4, VPP=12 | BF16/MXFP8 | HybridEP |

## 3. Recompute and CUDA Graph Strategy
For long context, the interaction between recompute and CUDA graphs is highly sensitive.

### Selective Recompute Priority
Prefer recomputing specific modules before falling back to full recompute:
- **High Priority**: `up_proj`, `norm`, `moe`, `mlp`.
- **Caution**: Recomputing attention internals (SDPA) at very long contexts can add massive compute overhead for marginal memory gain.

### CUDA Graph Guidelines
- **Staticity First**: Use CUDA graphs only after sequence length, MBS, and routing paths are 100% stable.
- **Scope**: Prefer TE-scoped graphs over full-iteration graphs to keep dynamic expert work outside the graph.

## 4. Critical Pitfalls
- **The "Fit-Only" Trap**: A config that "fits" in memory may have abysmal throughput. Always balance CP with the DP budget.
- **Baseline Drift**: A configuration that worked for 4K tokens is almost never a good starting point for 128K tokens.
- **Varying Batch Lengths**: Dynamic padding can silently break CUDA graphs, causing mysterious crashes or performance drops.
- **Hardware-Kernel Dependency**: 128K+ paths often rely on the absolute latest kernel versions; ensure the container is up-to-date.
