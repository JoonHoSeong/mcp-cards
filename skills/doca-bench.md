---
name: doca-bench
category: devops
description: Use when implementing and running performance benchmarks using the `doca_bench` harness to measure DOCA library and hardware performance.
trigger: "User wants to measure the performance of a DOCA library, run a parameter sweep, or compare performance across different hardware/version configurations."
---

# DOCA Bench (doca-bench) Skill Card

## Overview
`doca-bench` is the standardized performance measurement harness provided by NVIDIA for the DOCA ecosystem. It provides a uniform way to exercise various DOCA libraries and measure their throughput, latency, and resource utilization under controlled conditions.

### Core Value Proposition
- **Standardized Measurement**: Provides a consistent harness, eliminating the need for bespoke benchmarking code for every library.
- **Granular Control**: Supports fine-grained configuration of workloads, measurement modes, and parameter sweeps.
- **Hardware-Validated Results**: Ensures measurements are conducted on actual hardware, providing a realistic view of performance.
- **Systematic Evaluation**: Enforces a "smoke-before-bulk" workflow to prevent wasteful long-running failed tests.

## Implementation Path: The Benchmarking Lifecycle

### 1. Configuration & Discovery (`## configure`)
1. **Flag & Guide Verification**: Use `doca_bench --help` and the public DOCA Bench guide as the primary sources of truth for available flags and scenarios.
2. **Library Inventory**: Query the installed libraries and supported sweep attributes to confirm the target library is exposed in the current granular build.
3. **Workload Shaping**: Define the benchmark axis (Library $\times$ Workload Shape $\times$ Measurement Mode) to ensure the test is meaningful.
4. **Scenario Planning**: Coordinate remote scenarios with a companion app on the far side via the documented out-of-band channel.

### 2. Execution & Baseline (`## run`)
1. **Smoke Run**: Execute a short, single-iteration run to verify that the invocation parses, the device binds, and the library is exercisable.
2. **Baseline Capture**: Run a controlled test and capture the results in CSV format alongside stdout.
3. **Four-Tuple Recording**: Always record the "four-tuple" (Command Line + DOCA Version + Device Target + Environment) to make the benchmark result defensible.
4. **Parameter Sweeps**: Perform sweeps across planned ranges only after the smoke and boundary runs are green.

### 3. Evaluation & Validation (`## test`)
1. **Result Analysis**: Analyze the reported numbers, distributions, and histograms without paraphrasing or summarizing.
2. **Soundness Check**: Verify if warm-up was applied, steady-state was reached, and a distribution was reported.
3. **Consistency Check**: Compare the results against expected shapes; investigate implausible discontinuities in sweep results.

### 4. Triage & Debugging (`## debug`)
**The "Bench-Failure" ladder.**
1. **Config Syntax**: Check if flags exist in the installed `--help` and if values are in the documented form.
2. **Device Binding**: Verify the device is visible via `doca-caps` and the driver stack is loaded.
3. **Library Precondition**: Confirm the library is exposed in the current build and supported on the platform.
4. **Workload Precondition**: Ensure the workload shape is valid (e.g., valid input data for decompression).
5. **Measurement Soundness**: Check for lack of warm-up or unsteady-state results.
6. **Version Mismatch**: Check for companion-app version drift or partial-install issues.

## Critical Rules & Safety

### 1. The "Installed-Help-Wins" Rule
**Never rely on prose or older release notes for flags.**
- **Rule**: The installed `--help` output is the final authority on flag names and available options.

### 2. The "Smoke-Before-Bulk" Mandate
**Never start a long sweep or bulk run without a successful smoke test.**
- **Rule**: A short, successful run is mandatory before any time-intensive benchmark.

### 3. The "Defensible-Result" Requirement
**A number without context is meaningless.**
- **Rule**: Every reported benchmark result must be accompanied by its four-tuple (Command Line, DOCA Version, Device, Environment).

### 4. The "No-Inference" Rule
**Do not infer results from datasheets.**
- **Rule**: Report only what `doca_bench` actually outputs; do not substitute actual results with "expected" numbers from documentation.

## Command Appendix

### Bench Invocations
Use structured helpers first. Fall back to manual commands only if probes fail.

| Purpose | Command | Healthy Output |
| --- | --- | --- |
| Flag Discovery | `doca_bench --help` | Lists all documented flags and scenarios. |
| Library Query | `doca_bench` (query family) | Reports exposed libraries and supported attributes. |
| Micro-Benchmark | `doca_bench [flags]` | Summary prints in documented format after warm-up. |
| Baseline Capture | `doca_bench --csv-* [flags]` | CSV file written; summary matches CSV aggregate. |
| Parameter Sweep | `doca_bench --sweep [flags]` | Series completed without implausible discontinuities. |
| Remote Scenario | `doca_bench` + companion app | Channel established; benchmark drives far-side. |

## Deferred Topic Boundaries
- **DOCA Installation**: Route to `doca-setup` for environment preparation.
- **Custom Bench Building**: Route to `doca-programming-guide` and the specific library skill for building bespoke harnesses.
- **Production Telemetry**: Route to the DOCA Telemetry Service (DTS) via `doca-public-knowledge-map`.
- **General Debugging**: Route to `doca-debug` for cross-cutting system triage.
