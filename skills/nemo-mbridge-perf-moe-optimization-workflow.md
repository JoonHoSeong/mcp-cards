---
name: nemo-mbridge-perf-moe-optimization-workflow
description: Systematic workflow for MoE training optimization in Megatron Bridge, based on the Megatron-Core MoE paper. Covers the Three Walls framework, parallel folding, recompute strategy, dispatcher choice, and CUDA-graph bring-up.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE Training Optimization Workflow

This workflow follows the "Three Walls" framework derived from the *Scalable Training of MoE Models with Megatron Core* research, designed to move systematically from "fitting" to "peak throughput."

## 1. The Three Walls Framework
MoE tuning is an iterative process. Fixing one bottleneck usually exposes the next.

- **The Memory Wall**: The run OOMs or requires aggressive full recompute.
- **The Communication Wall**: Nsight profiles show dominant `all-to-all` or collective blocks.
- **The Compute/Host Wall**: GPU gaps are visible; run is launch-bound or Python-overhead bound.

## 2. The Systematic Optimization Pipeline

### Phase 1: Make the Run Memory-Feasible (Fit)
*Goal: Get the model to run without OOM using the least aggressive memory settings.*
1. **Minimize Parallelism**: Use the smallest TP/PP that fits.
2. **Selective $\rightarrow$ Full Recompute**: Try selective recompute (e.g., `moe_act`, `mlp`) before blanket full recompute.
3. **Offload as Last Resort**: Add CPU offloading for optimizer states only when parallelism and recompute are insufficient.
4. **Sanity Check**: Use `--fake-init-process-group` to validate huge layouts on a single GPU before cluster launch.

### Phase 2: Maximize Scale (Scale)
*Goal: Increase throughput by optimizing the data-parallel (DP) budget.*
1. **Maximize DP**: Once the model fits, increase DP as much as possible.
2. **Interconnect Affinity**: Keep hot communication (TP/EP) inside the fastest interconnect (NVLink).
3. **PP + VPP**: Use Pipeline Parallelism with Virtual Pipeline stages for multi-node scaling.
4. **Prefer EP over TP**: For expert layers, prioritize EP to keep the expert count high without inflating TP.
5. **Add CP**: Introduce Context Parallelism only when sequence length makes attention memory the primary bottleneck.

### Phase 3: Identify & Fix the Dominant Bottleneck (Profile & Retune)
*Goal: Move from "running" to "optimized" by targeting the specific wall.*

| Bottleneck | Profile Signal | Primary Fix |
|---|---|---|
| **Memory** | OOM during warmup, high recompute overhead | Selective recompute, FP8, better PP layout |
| **Communication** | Large `all-to-all` blocks in Nsight | DeepEP/HybridEP, EP overlap, DP/TP overlap |
| **Host/Launch** | Large GPU gaps, launch-bound traces | CUDA graphs, `--manual-gc`, higher MBS |
| **Compute** | Low SM utilization after comm is fixed | Grouped GEMM, fusion work, dispatcher kernel tuning |

## 3. Advanced Configuration Knobs

### Parallel Folding
Decouple attention and MoE parallelism to avoid a "one-size-fits-all" compromise:
- **Attention Mesh**: `TP x CP x DP x PP`
- **MoE Mesh**: `ETP x EP x EDP x PP`
- *Key Knobs*: `--expert-model-parallel-size`, `--expert-tensor-parallel-size`

### Dispatcher Selection
- **`alltoall`**: Safest for bring-up; use for small EP.
- **`flex` + `deepep`**: High-performance default for H100/B200.
- **`flex` + `hybridep`**: Required for GB200/GB300 NVL72 systems.

### Precision & CUDA Graphs
- **Precision**: Hopper $\rightarrow$ FP8 blockwise; Blackwell $\rightarrow$ MXFP8. Keep the router in FP32.
- **CUDA Graphs**: Use TE-scoped graphs (`attn`, `moe_router`, `moe_preprocess`) for dropless MoE.

## 4. Critical Pitfalls
- **Wrong Order**: Never try to tune the dispatcher before the model fits and the DP budget is maximized.
- **Platform Drift**: H100 runs are often comm-bound; GB200 runs often expose host/launch overhead earlier.
- **MFU Deception**: When switching precision, compare absolute TFLOPS. MFU can be misleading across precision modes.
- **Graph/Recompute Clash**: TE-scoped graphs typically pair with selective recompute, not blanket full recompute.
