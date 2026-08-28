---
name: doca-flow-perf
category: devops
description: Measure the performance of the DOCA Flow pipeline (insertion rate, cycles per operation) on the host or DPU-CPU path using the `doca_flow_perf` tool.
trigger: "User wants to measure the performance of a specific DOCA Flow pipeline configuration, compare insertion rates across backends, or verify if a Flow policy meets performance targets."
---

# DOCA Flow Perf (doca-flow-perf) Skill Card

## Overview
`doca_flow_perf` is a specialized tool for measuring the performance of the DOCA Flow pipeline. It focuses on the synthetic measurement of insertion rates and cycles per operation, allowing developers to benchmark their pipeline configurations before deploying them in a live application.

### Core Value Proposition
- **Synthetic Benchmarking**: Measures the performance of the Flow pipeline without needing a full application.
- **Backend Comparison**: Compare performance between the DPDK and DOCA backends.
- **Policy-Driven Workloads**: Use JSON policies to define the matchers and actions being benchmarked.
- **Variance Analysis**: Provide a disciplined approach to measuring performance across multiple iterations to ensure results are defensible.

## Implementation Path: The Perf Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Library Verification**: Confirm the `doca-flow` library version matches the system's `doca_caps` version.
2. **Tool Discovery**: Verify the `doca_flow_perf` binary and the `configs/` JSON library are present.
3. **Policy Selection**: Identify the canned JSON policy from the `configs/` directory that most closely matches the intended workload.
4. **Capability Match**: Cross-reference the matchers and actions in the JSON policy against the target device's `doca_caps` output to ensure compatibility.
5. **Workload Sizing**: Define the `num_inserted_entries` and iteration count based on the device's table limits and the required statistical confidence.
6. **Backend Choice**: Select the backend (DPDK or DOCA) for the measurement.

### 2. Execution & Run (`## run`)
1. **Single-Iteration Smoke**:
   - **Rule**: Mandatory. Run a single iteration with a low entry count to verify the JSON policy is accepted and the pipeline is created successfully.
2. **Scaled Measurement**:
   - **Rule**: Run the full workload size for N iterations.
   - **Data Capture**: Capture the raw per-iteration cycles and counters verbatim.
3. **Result Synthesis**:
   - **Rule**: Report the backend (DPDK/DOCA) and the "Four-Tuple" (DOCA version, BlueField generation, firmware version, JSON policy).

### 3. Verification & Analysis (`## test`)
1. **Clean Run Check**: Verify that `num_pushed` equals `num_inserted_entries`. Any `num_failed > 0` is a finding (resource exhaustion) rather than a result.
2. **Variance Computation**: Compute the mean and standard deviation across iterations off-line. Do not silently average high-variance results.
3. **Defensibility Check**: Confirm that the reported number is compared only against others with a matching Four-Tuple.

## Critical Rules & Safety

### 1. The "Smoke-before-Bulk" Mandate
**Scaled measurements must be preceded by a clean single-iteration smoke test.**
- **Rule**: Running `doca_flow_perf` at scale can exhaust device Flow tables and disrupt co-tenant applications. A clean smoke test is non-negotiable.

### 2. The "Four-Tuple" Requirement
**A performance number without its context is not actionable.**
- **Rule**: Every reported result must include: (1) DOCA version, (2) BlueField/ConnectX generation, (3) Firmware version, and (4) the JSON policy content.

### 3. The "Backend-Explicit" Rule
**The backend choice (DPDK vs DOCA) fundamentally changes the result.**
- **Rule**: Every reported number must explicitly state which backend produced it.

### 4. "Quote, Do Not Paraphrase"
**Fidelity is critical for performance data.**
- **Rule**: Quote the raw per-iteration output and the JSON policy verbatim. Paraphrasing leads to lost fidelity and incorrect analysis.

## Diagnostic Ladder (Overlay)

| Layer | Perf Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Install)** | Binary or `configs/` not found. | Route to `doca-setup`. |
| **Layer 2 (Policy)** | JSON parse error or malformed policy. | Compare against shipped `configs/` exemplars; do not invent keys. |
| **Layer 3 (Pipeline)** | Pipeline creation failed (e.g., unsupported matcher). | Route to `doca-flow` debug ladder. |
| **Layer 4 (Resources)** | `num_failed > 0` during insertion. | Reduce entry count or check device table limits. |
| **Layer 5 (Variance)** | High variance across iterations. | Pin worker threads, isolate CPUs, and increase iteration count. |
| **Layer 6 (Version)** | Disagreement with reference numbers. | Walk the `doca-version` debug ladder. |

## Command Appendix

### Flow Perf Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Purpose | Command (Class Shape) | Owning Step | Healthy Output |
| --- | --- | --- | --- |
| CLI Surface Discovery | `doca_flow_perf --help` | `## configure` | Documented flag inventory. |
| Policy Discovery | Inspect `configs/` subdirectory | `## configure` | Identification of suitable canned policy. |
| Library Version | `pkg-config --modversion doca-flow` | `## configure` | Matches `doca_caps --version`. |
| Device Capability | `doca_caps` probes | `## configure` | Matchers/actions in JSON are supported. |
| Smoke Test | `doca_flow_perf` (low entries, 1 iter) | `## run` | Exit 0; `num_failed == 0`. |
| Scaled Run | `doca_flow_perf` (full workload, N iters) | `## run` | Exit 0; per-iteration cycles captured. |
| Variance Analysis | Manual calculation of mean/std-dev | `## test` | Defensible result with low variance. |
| Session Snapshot | Capture Policy + Command + Output + Four-Tuple | `## test` | Verbatim bundle for downstream debug. |

## Deferred Topic Boundaries
- **Application Code**: Modifying a `doca-flow` app is handled in `doca-flow`.
- **Optimization**: Optimizing a live app is handled in `doca-flow-tune`.
- **DPA Offload**: Measuring DPA-offloaded paths is handled in `doca-flow-dpa-perf`.
- **End-to-End Throughput**: Measuring actual packet throughput is the application's responsibility, layered on `doca-flow`.
- **Environment**: Firmware/driver updates are handled in `doca-setup` and `doca-version`.
