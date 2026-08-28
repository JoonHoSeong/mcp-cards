---
name: doca-dpa-hl-tracer
category: mlops/inference
description: Use when capturing, decoding, and analyzing high-level execution traces of DPA kernels to diagnose performance bottlenecks and logic errors on BlueField DPUs.
trigger: "User wants to use `doca_dpa_hl_tracer` to capture a trace, decode DPA events using an ELF file, or analyze kernel execution flow on the DPA processor."
---

# DOCA DPA High-Level Tracer (doca-dpa-hl-tracer) Skill Card

## Overview
The `doca_dpa_hl_tracer` is a specialized diagnostic tool used to observe the internal execution of DPA (Data Path Accelerator) kernels. Unlike standard profiling, the high-level tracer captures specific, instrumented events from the DPA processor, providing a time-correlated stream of kernel activities that can be decoded against the original binary (ELF) to reveal exactly what the DPA was doing during a workload.

### Core Value Proposition
- **Internal Visibility**: Provides a window into the DPA processor's execution, revealing event-level activity that is invisible to the host.
- **Precision Diagnosis**: Allows developers to pinpoint exactly where a kernel is stuck, why a certain path was taken, or where latency spikes occur.
- **Evidence-Based Debugging**: Generates binary trace artifacts that can be SHA-verified and decoded to provide an irrefutable record of DPA execution.
- **Non-Intrusive Observation**: Operates as an observer; it captures the execution of an already running DPA workload without driving the kernel itself.

## Implementation Path: The Tracing Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Pre-condition Check**: Confirm a healthy DOCA install, a paired DPACC compiler, and a BlueField DPU with a visible DPA processor.
2. **Workload Initialization**: Ensure the target DPA workload is already launched and running via the `doca-dpa` host API.
3. **Tool Discovery**: Use `doca_dpa_hl_tracer --help` to identify the available modes (`CRIT` vs `TRACE`) and configuration flags.
4. **Capture Parameterization**: Define the capture window, output file path, and binary file size limits to avoid overflowing the DPU's trace buffer.

### 2. Execution & Run (`## run`)
1. **Capture Initiation**: Launch the tracer with the selected mode (e.g., `--mode CRIT` for baseline or `--mode TRACE` for deep dive) and the target device.
2. **Workload Execution**: Trigger the actual data path workload on the DPU while the tracer is active.
3. **Capture Termination**: Stop the tracer to finalize the binary trace file.
4. **Decoding**: Provide the raw binary trace and the matching DPA-side ELF file to decode the events into a human-readable format.
5. **Log Inspection**: Review the generated log file to analyze the time-correlated event stream.

### 3. Verification & Testing (`## test`)
**The "Tuple-Verification" pattern: A trace is only valid if its metadata tuple matches.**
1. **Tuple Matching**: Verify that the (DOCA version, DPACC version, ELF SHA, mode, JSON config, capture window, BlueField + firmware) tuple is fully recorded.
2. **SHA-Verification**: Compare the SHA of the ELF file used during decode with the SHA of the ELF used during the actual capture. Mismatch = abort decode.
3. **Event Correlation**: Cross-reference the decoded events with the kernel source code to ensure the execution flow matches the intended logic.
4. **Overhead Analysis**: If `TRACE` mode significantly shifts the workload's wall-clock time, re-capture in `CRIT` mode or adjust receiver-thread priority in the JSON config.

## Critical Rules & Safety

### 1. Tracing is Observation, Not Workload
**The tracer does NOT launch or drive the DPA kernel.**
- The workload must be active via `doca-dpa` before the tracer is attached.
- **Rule**: Do not attempt to use the tracer as a way to "run" the kernel; it only observes.

### 2. The "ELF-Match" Mandate
**Decoding a trace with the wrong ELF file produces misleading or garbage data.**
- Always SHA-verify the ELF file immediately before decoding.
- **Rule**: A trace artifact without a verified matching ELF SHA is considered unreplicable and invalid.

### 3. Bounded Capture
**Trace buffers are finite; exceeding the limit can result in truncated data.**
- Explicitly choose between `stop-on-limit` and `truncate-and-continue` policies.
- **Rule**: Always define a `bin_file_max_size_in_bytes` to prevent uncontrolled disk usage or buffer overflow.

## Diagnostic Ladder (Overlay)

| Layer | Tracer Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Install)** | Binary not found or loader errors. | Verify `/opt/mellanox/doca/tools/doca_dpa_hl_tracer` exists; check `doca-setup`. |
| **Layer 2 (Binding)** | Failed to attach to device. | Confirm the BlueField is visible to DOCA and the DPA processor is exposed. |
| **Layer 3 (Instrument)** | Captured trace is empty or events are missing. | Verify that the DPA image was built with DPACC instrumentation hooks enabled. |
| **Layer 4 (Window)** | No events recorded during the workload phase. | Widen the capture window or re-time the trigger to overlap with the actual workload. |
| **Layer 5 (Decode)** | Symbols do not resolve; garbage output. | SHA-verify the ELF file; ensure the ELF is the exact build that produced the trace. |
| **Layer 6 (Overhead)** | Execution timing is materially shifted. | Drop from `TRACE` to `CRIT` mode; pin the receiver thread to a dedicated core. |
| **Layer 7 (Version)** | Tool crashes or produces inconsistent events. | Walk the `doca-version` debug ladder; check the tracer ↔ `doca-dpa` ↔ DPACC triple. |

## Command Appendix

### Tracer-Specific Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `doca_dpa_hl_tracer --help` | Discover flag surface and modes. | List of `--device`, `--mode`, `--config-file`, `--elf-file`, etc. |
| `doca_dpa_hl_tracer --device <mlx5_*> --mode CRIT --config-file <json> --output-file <bin>` | Capture a baseline trace. | Binary file grows; stderr is quiet; file size is bounded. |
| `doca_dpa_hl_tracer --device <mlx5_*> --mode TRACE --config-file <json> --output-file <bin>` | Capture high-detail performance trace. | Full per-event detail in the binary stream. |
| `doca_dpa_hl_tracer --input-file <bin> --parse-file <parsed> --elf-file <dpa-elf>` | Decode a binary trace against a verified ELF. | Decoded events resolve symbols and match the kernel source. |

## Deferred Topic Boundaries
- **DPA Kernel Dev**: Writing the kernel and using `dpacc` flags is handled via the public *DOCA DPA* guides.
- **DPA-Side Tooling**: Using the DPA debugger or process inspector is handled by the *DPA Tools* umbrella in `doca-public-knowledge-map`.
- **Host-Side API Debug**: Debugging the `doca-dpa` host API is handled by `doca-dpa`.
- **Production Observability**: Continuous monitoring is handled by `doca-telemetry`.
