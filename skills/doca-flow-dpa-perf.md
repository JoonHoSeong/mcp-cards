---
name: doca-flow-dpa-perf
category: devops
description: Use the `doca_flow_dpa_perf` tool to measure and baseline the performance of the DPA (Data Path Accelerator) path for DOCA Flow workloads.
trigger: "User wants to measure DPA-specific flow performance, conduct workload-shape sweeps, verify DPA resource allocation, or baseline DPA throughput/latency against a specific DOCA version and device."
---

# DOCA Flow DPA Perf (doca-flow-dpa-perf) Skill Card

## Overview
`doca_flow_dpa_perf` is a specialized performance measurement harness used to isolate and measure the performance of the DPA (Data Path Accelerator) execution path for DOCA Flow workloads. Unlike host-side performance tools, this tool directly exercises the DPA hardware to provide an accurate baseline of what the DPA path can deliver.

### Core Value Proposition
- **DPA Path Isolation**: Isolates the DPA execution path from host-side bottlenecks, providing a pure hardware baseline.
- **Workload-Shape Analysis**: Allows for sweeping across various parameters (burst size, queue depth, workers) to identify the optimal configuration for a given workload.
- **Regression Testing**: Enables precise baseline capture (using the four-tuple) to detect performance regressions across DOCA versions or firmware updates.
- **Soundness Verification**: Uses iteration statistics and self-tests to ensure that the measured numbers are defensible and not artifacts of noise or improper configuration.

## Implementation Path: The Measurement Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Source of Truth Alignment**: Align with the shipped `README.md`, installed `--help`, and the public DOCA Flow DPA Perf guide. Resolve defaults in the order: README $\rightarrow$ `--help` $\rightarrow$ User request.
2. **Hardware Verification**: Confirm the device is ConnectX-7+ or BlueField-3 (DPA-capable). Verify that the device is not a SF (Smart-NIC) and is in VNF Flow mode.
3. **DPA-Precondition Check**: Confirm that the required DPA execution resources are available and the Eswitch/LAG configuration matches the requirements in the README.
4. **Tool Discovery**: Use `doca_flow_dpa_perf --help` to identify available flags and workload-shape parameters.

### 2. Run & Execution (`## run`)
1. **Smoke Run**: Execute a single-worker update with small `num-operations` and `num-iterations` to ensure the path is healthy. **Smoke-before-bulk is mandatory.**
2. **Baseline Measurement**: Run the tool with warmup enabled and sufficient iterations to ensure a stable median.
3. **Workload Sweep**: Perform sweeps across planned axes (e.g., burst size, queue depth) to identify performance trends and optimal points.
4. **Self-Test Correctness**: Run the tool's self-test and verify the path-selector sentinel `65432` using `tcpdump` to ensure the DPA is processing packets correctly.
5. **Verification**: Quote the full iteration-stats block, including median and standard deviation, to ensure measurement soundness.

### 3. Baseline Capture & Testing (`## test`)
**The "Four-Tuple" Baseline Rule.**
To make a measurement meaningful, every captured baseline must include:
- **Command Line**: The exact invocation used.
- **DOCA Version**: The version of the DOCA SDK installed.
- **Device**: The specific PCIe device (and class) used.
- **As-Deployed Environment**: The environment configuration.
- **Tool Name**: Explicitly state `doca_flow_dpa_perf`.

If any of these are missing, the measurement is considered unfalsifiable and cannot be used for regression testing.

### 4. Triage & Debugging (`## debug`)
**The "DPA-Perf" Debug Ladder.**
1. **Config-Syntax**: Verify flags against `--help` and the README.
2. **Device-Binding**: Confirm PCI addresses via `doca-caps`.
3. **DPA-Precondition**: Verify VNF Flow mode and DPA resources. Route to `doca-dpa` for resource errors.
4. **Workload-Precondition**: Verify burst-size divisibility, queue-size power-of-2, and worker ranges.
5. **Measurement-Soundness**:
   - **High Variance**: Increase iterations or check for background noise.
   - **Median $\neq$ Max**: Re-examine warmup and iteration strategy.
6. **Self-Test Failure**: Re-walk the README's self-test section and verify the sentinel.
7. **Version**: Route to `doca-version` if numbers shift across versions.
8. **Cross-Cutting**: Route to `doca-debug` or `doca-setup` for systemic failures.

## Critical Rules & Safety

### 1. Hardware-Driving Tool
**`doca_flow_dpa_perf` drives hardware and allocates DPA resources.**
- **Rule**: Always perform a smoke run before executing bulk measurements to avoid system instability.

### 2. The "Four-Tuple" Mandate
**Measurements without the four-tuple are meaningless.**
- **Rule**: The agent must either capture or request the four-tuple (Command + Version + Device + Environment) alongside the tool name for every reported baseline.

### 3. No-Inference Rule
 **Do not infer missing defaults.**
- **Rule**: If the README and `--help` are silent on a default value, the agent must stop and request it from the user.

### 4. Cross-Tool Comparison Warning
**DPA-perf numbers are not directly comparable to host-side Flow-perf numbers.**
- **Rule**: If the user compares DPA-perf to `doca-flow-perf`, the agent must explicitly state that they measure different paths.

## Command Appendix

### DPA-Perf Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `doca_flow_dpa_perf --help` | Discover flags and documented surface. | List of available flags and workload parameters. |
| `doca_flow_dpa_perf --device <mlx5_*> --num-operations <N> --num-iterations <M>` | Smoke run / baseline measurement. | Stable iteration-stats block with low standard deviation. |
| `doca_flow_dpa_perf --self-test` | Verify DPA processing correctness. | Path-selector sentinel `65432` observed on the wire. |
| `doca_flow_dpa_perf <sweep-flags>` | Sweep across a workload-axis. | A series of results without implausible discontinuities. |

## Deferred Topic Boundaries
- **Host/DPU-CPU Flow Path**: Measured via `doca-flow-perf`.
- **Pipeline Optimization**: Performed via `doca-flow-tune` after a sound baseline is established.
- **DOCA Installation**: Handled via `doca-setup`.
- **Application Development**: Writing bespoke Flow/DPA apps is handled by `doca-flow` and `doca-dpa` under the `doca-programming-guide`.
- **Live Metrics/Telemetry**: Handled by the DOCA Telemetry Service (DTS) via `doca-public-knowledge-map`.
