---
name: doca-erasure-coding
category: mlops/inference
description: Use when implementing data redundancy and recovery using the DOCA Erasure Coding (EC) library for high-availability storage systems on BlueField DPUs.
trigger: "User wants to perform data erasure coding (create parity, recover missing blocks, or update parity) using DOCA hardware acceleration."
---

# DOCA Erasure Coding (doca-erasure-coding) Skill Card

## Overview
The DOCA Erasure Coding (EC) library provides hardware-accelerated implementation of erasure coding primitives. It allows the offloading of computationally expensive parity calculations (encoding) and data recovery (decoding) to the BlueField DPU, reducing host CPU overhead and increasing throughput for distributed storage systems.

### Core Value Proposition
- **High-Throughput Parity**: Offloads N+K parity calculations to DPU hardware.
- **Fast Data Recovery**: Accelerates the recovery of missing data blocks from parity and surviving data blocks.
- **Efficient Updates**: Supports updating parity without re-encoding the entire data set when a single block changes.
- **Reduced Host Latency**: Minimizes the CPU cycles spent on Reed-Solomon or similar EC algorithms.

## Implementation Path: The EC Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Version Verification**: Perform a three-way version match (pkg-config, `doca_caps`, and install tree) to ensure the EC library is correctly installed.
2. **Device Capability Query**: Use `doca_ec_cap_*` functions to determine the supported task types, maximum block sizes, and supported matrix variants for the active `doca_dev`.
3. **Matrix Selection**: Choose a supported matrix type (e.g., Reed-Solomon) and define the N (data) and K (parity) layout based on the device's maximum buffer list length.
4. **Context Initialization**: Initialize the DOCA context and enable the required EC tasks before calling `doca_ctx_start()`.

### 2. Execution & Run (`## run`)
1. **Task Submission**: Submit EC tasks (Create, Recover, or Update) using `doca_ec_task_submit()`.
2. **Buffer Management**: Ensure all source and destination buffers are allocated via `doca_mmap` and have lengths exactly matching the configured block size.
3. **PE Progression**: Call `doca_pe_progress()` in the main loop to drain the completion queue and receive the results of submitted tasks.
4. **Result Verification**: Confirm that the recovered data matches the original or that the generated parity is correct.

### 3. Verification & Testing (`## test`)
**The "Sizing-vs-Cap" loop.**
1. **Small-Bulk Smoke**: Test with a small dataset to verify the basic Create $\rightarrow$ Recover $\rightarrow$ Update flow.
2. **Boundary Testing**: Intentionally test block sizes and N+K layouts at the limits of the device's reported capabilities.
3. **Negative Testing**: Request unsupported task types or invalid block sizes to confirm the library returns `DOCA_ERROR_NOT_SUPPORTED` or `DOCA_ERROR_INVALID_VALUE`.
4. **Scale-Up Performance**: Increase the submission rate and verify that the PE is progressed fast enough to prevent `DOCA_ERROR_AGAIN`.

## Critical Rules & Safety

### 1. The "Exact-Block-Size" Mandate
**Every buffer in an EC task must have a length exactly equal to the configured block size.**
- **Rule**: Mismatched buffer lengths will result in `DOCA_ERROR_INVALID_VALUE` at submission. Do not attempt to pass buffers of varying sizes within a single task.

### 2. The "PE-Progress" Requirement
**Tasks are asynchronous; they will never complete without an explicit progress call.**
- **Rule**: `doca_pe_progress()` must be called frequently. If `doca_task_submit()` returns success but no completion event appears, the PE is not being progressed.

### 3. The "N+K Budget" Limit
**The total number of buffers (N+K) cannot exceed the device's maximum buffer list length.**
- **Rule**: Always check `doca_ec_cap_get_max_buf_list_len()` before defining the EC layout. Exceeding this limit is a hard failure.

## Diagnostic Ladder (Overlay)

| Layer | EC Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Install)** | `pkg-config` fails or version mismatch. | Verify the install tree and `doca_caps --version`. |
| **Layer 2 (Version)** | Matrix variant not supported by the current device. | Re-enumerate supported matrix types via `doca_ec_matrix_create()` and the active `doca_devinfo`. |
| **Layer 3 (Runtime)** | `DOCA_ERROR_BAD_STATE` during submit. | Confirm the context is started and the specific EC task was enabled before `doca_ctx_start()`. |
| **Layer 4 (Program)** | `DOCA_ERROR_INVALID_VALUE` during submit. | Verify that all buffer lengths match the configured block size and N+K is within limits. |
| **Layer 5 (Recover)** | Recovered data is incorrect or length is wrong. | Check for matrix type mismatch, N+K mismatch, or source/destination buffer swap. |
| **Layer 6 (Driver)** | `DOCA_ERROR_DRIVER` or repeated accelerator errors in `dmesg`. | Capture `dmesg` and `mlxconfig` output; route to `doca-setup`. |

## Command Appendix

### EC-Specific Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `pkg-config --modversion doca-erasure-coding` | Library version verification. | Semver matching `doca_caps --version`. |
| `pkg-config --cflags --libs doca-erasure-coding` | Build-time flags for the linker. | Trust `pkg-config` output; do not hardcode `-l` flags. |
| `doca_caps --list-devs` | Device compatibility check. | List of devices with EC capability flags. |
| `ls /opt/mellanox/doca/samples/doca_erasure_coding/` | Sample application discovery. | Directories for create, recover, and update samples. |
| `DOCA_LOG_LEVEL=trace ./<binary>` | EC lifecycle and submission trace. | Trace-level lines for every lifecycle transition and task submission. |

## Deferred Topic Boundaries
- **Storage Stack Design**: Designing the overall redundancy strategy (e.g., layering replication on top of EC) is outside this skill.
- **Deployment at Scale**: Managing EC workloads across a cluster of DPUs is reserved for future platform skills.
- **Firmware/Driver Install**: Modifying `mlxconfig` or burning firmware is handled by `doca-setup`.
- **General Memory Management**: Using `doca_mmap` for buffer allocation is handled by `doca-common`.
