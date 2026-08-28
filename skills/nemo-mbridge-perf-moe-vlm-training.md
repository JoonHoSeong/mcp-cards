---
name: nemo-mbridge-perf-moe-vlm-training
description: Practical guidance for training MoE VLMs in Megatron Bridge. Compares FSDP and 3D-parallel approaches, using rounded lessons from Qwen3-VL, Qwen3-Next, and other multimodal experiments.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE VLM Training Guide

Training Mixture-of-Experts Vision-Language Models (VLMs) introduces unique challenges, primarily around the interaction between the vision encoder, projector, and MoE decoder.

## 1. FSDP vs. 3D Parallelism
The first critical decision is choosing the parallelism strategy.

### Strategy Comparison
| Approach | Strength | Best Fit |
|---|---|---|
| **FSDP** | Fastest time-to-run, simplest setup | Initial bring-up, memory-constrained runs, awkward PP boundaries |
| **3D Parallel** | Highest steady-state throughput | Stable models with clean PP layouts, production-scale training |

**Practical Workflow**: Start with **FSDP** to stabilize the real-data pipeline $\rightarrow$ Tune memory/recompute $\rightarrow$ Migrate to **3D Parallel** once the throughput headroom justifies the complexity.

## 2. Key Multimodal Findings (Lessons from Qwen3-VL/Next)

### The "Mock Data" Trap
**Critical**: Never use image-free mock data for performance profiling. 
- Mock runs can be $\sim 2\times$ faster than real multimodal runs.
- Real image payloads change the compute-to-overhead balance and can reveal hidden bottlenecks in the vision encoder.

### Hardware-Specific Patterns
- **GB200/GB300**: HybridEP is the strong default. FSDP can already reach high-teens utilization with minimal tuning.
- **B200**: FSDP is viable but highly sensitive to the choice of recompute and whether the vision stack is frozen.

## 3. Optimization & Tuning Knobs

### Vision Stack Management
- **Freeze the Encoder**: If the task is decoder-focused, freezing the vision stack reduces memory pressure and provides a modest throughput gain.
- **MBS Sweeping**: VLMs are more sensitive to Micro-Batch Size (MBS) than text-only models. Aggressively sweep MBS to find the balance between the vision encoder and MoE decoder.

### Memory and Compute
- **Recompute**: Start with full recompute for bring-up, then relax toward **Selective Recompute** for steady-state.
- **CUDA Graphs**: Use `attn moe_router moe_preprocess` as the safe MoE default. Only widen the scope after the real-data path is stable.
- **ETP**: Use Expert Tensor Parallelism (ETP) only as a tool to make a layout fit; it adds communication overhead.

## 4. Representative Configuration Families

### FSDP-First Path (GB200)
- **Parallelism**: TP=1, CP=1, PP=1. EP sized to expert topology.
- **Dispatcher**: HybridEP.
- **Recompute**: Full $\rightarrow$ Selective.

### 3D-Parallel Path (GB200)
- **Parallelism**: TP=1, CP=1, PP=1 (or modest PP). EP and ETP sized to topology.
- **Dispatcher**: HybridEP.
- **CUDA Graph**: Start narrow, widen after stability.

## 5. Critical Pitfalls
- **Encoder Bottlenecks**: The vision encoder can unexpectedly dominate the runtime. Profile the encoder, projector, and decoder separately.
- **Normalization**: When comparing FSDP vs 3D Parallel, normalize by **useful tokens**, not just step time, to account for different workload shapes.
- **Recompute/Graph Coupling**: The memory-fitting recompute setting is often different from the throughput-optimal setting.
