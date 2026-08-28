---
name: nemo-mbridge-perf-moe-dispatcher-selection
description: Selection guide for MoE token dispatchers (alltoall, DeepEP, HybridEP) based on hardware, EP degree, and optimization stage.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# MoE Dispatcher Selection Guide

Choosing the right dispatcher is critical for MoE throughput and memory headroom, especially on Hopper and Blackwell architectures.

## 1. Decision Matrix

### By Hardware Platform
| Hardware | First Choice | Rationale |
|---|---|---|
| H100 | **DeepEP** | Strongest default for cross-node EP on Hopper |
| B200 | **DeepEP** | Solid choice unless platform-specific HybridEP is available |
| GB200 / GB300 NVL72 | **HybridEP** | Best fit for NVLink-domain-aware dispatch and memory efficiency |
| First Bring-up | **`alltoall`** | Easiest path for correctness and debugging |

### By EP Degree
- **Small EP**: Dispatcher choice is second-order; start with `alltoall` or DeepEP.
- **Medium EP**: DeepEP becomes significantly more worthwhile.
- **Large EP**: HybridEP is the target for NVL72 systems.

## 2. Implementation Patterns

### DeepEP Activation
```bash
# Config
moe_token_dispatcher_type="flex"
moe_flex_dispatcher_backend="deepep"
# Tuning: SM count for comm kernels (default 20)
--moe-deepep-num-sms 20
```

### HybridEP Activation
```bash
# Config
moe_token_dispatcher_type="flex"
moe_flex_dispatcher_backend="hybridep"
# Tuning: SM count for comm kernels (default 16, harness uses 32)
--moe-hybridep-num-sms 32
# Topology: Must match NVLink domain size
export NUM_OF_HYBRID_EP_RANKS_PER_NVLINK_DOMAIN=<size>
```

## 3. Critical Pitfalls
- **Availability Gate**: Setting the backend flag alone is insufficient. If the package (DeepEP/HybridEP) is missing from the container, the run will fail during model construction.
- **Topology Sensitivity**: HybridEP is not a universal win; it is strictly designed for specific NVL72-class topologies.
- **SM Tuning**: Default SM counts are rarely optimal; always sweep between 16 and 32 for flex dispatchers.
- **Routing Baseline**: Always keep the routing mode (e.g., `--moe-router-force-load-balancing`) fixed when comparing dispatchers.

## 4. Model Family Examples (Reference)
- **DSV3**: HybridEP on GB200, DeepEP on H100.
- **Qwen3 235B**: HybridEP on GB200 often wins on memory and throughput.
- **Qwen3-Next**: HybridEP is stronger in FP8 or memory-tight runs.
