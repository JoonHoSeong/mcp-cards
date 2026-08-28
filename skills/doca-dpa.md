---
name: doca-dpa
category: mlops/inference
description: Use when implementing and managing DPA (Data Path Accelerator) kernels for high-performance, low-latency data processing on BlueField DPUs.
trigger: "User wants to write, compile, launch, or debug a DPA kernel, use DPA-side verbs, or optimize data path processing using the DPACC compiler."
---

# DOCA DPA (doca-dpa) Skill Card

## Overview
The DOCA DPA (Data Path Accelerator) API allows developers to offload complex data processing tasks from the CPU to the DPU's dedicated DPA processors. By writing kernels that run directly on the DPA, developers can achieve extreme throughput and near-deterministic latency for packet processing, encryption, and data transformation.

### Core Value Proposition
- **Deterministic Latency**: Bypasses the host CPU entirely for the data path, eliminating jitter and OS overhead.
- **Parallel Processing**: Leverages the DPA's specialized architecture for high-parallelism data movement and transformation.
- **Hardware Acceleration**: Direct access to DPU hardware resources via DPA-side verbs.
- **Specialized Toolchain**: Uses the `dpacc` compiler to translate high-level kernels into DPA-executable binary images.

## Implementation Path: The DPA Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Capability Verification**: Use `doca_caps --list-devs` to ensure the BlueField DPU supports DPA.
2. **Version Alignment**: Verify that the `doca-dpa` library, `dpacc` compiler, and BlueField firmware are all version-compatible per the DOCA Compatibility Policy.
3. **Resource Mapping**: Define the memory regions and Queue Pairs (QP) the DPA kernel will use for communication.
4. **Transport Selection**: Choose the transport (IB / RoCE) for the QP used by the DPA kernel.

### 2. Execution & Run (`## run`)
1. **Kernel Compilation**: Use `dpacc` to compile the DPA-side translation unit into a binary image.
2. **Host-Side Launch**: Use the `doca-dpa` host API to load and launch the kernel image on the DPA processor.
3. **Resource Passing**: Pass QP handles and buffer addresses as launch arguments to the kernel.
4. **Completion Handling**: Set up a completion surface (e.g., host-side CQE inspection) to monitor the kernel's progress.

### 3. Verification & Testing (`## test`)
**Follow the "One Launch, One Post, One Completion" smoke test pattern.**
1. **Single-Op Smoke**: Launch the kernel once, post exactly one Work Request (WR), and verify exactly one completion.
2. **Verb Verification**: Confirm the specific DPA-side verb used in the kernel is supported by the hardware via `doca_dpa_cap_is_supported`.
3. **Coupling Check**: Ensure the host-configured QP is capable of honoring the WR the DPA kernel posts.
4. **Scaling**: Only after the smoke test is green, move to streaming-launch patterns or bulk data processing.

## Critical Rules & Safety

### 1. Two-Side Program Rule
**A DPA application is a two-sided program; the host and the DPA kernel must be perfectly synchronized.**
- Any change to the QP configuration, buffer layout, or launch arguments must be applied to BOTH the host code and the DPA kernel.
- **Rule**: Rebuild BOTH sides via `dpacc` and the host compiler; never perform a partial rebuild.

### 2. DPA-Side Verbs Isolation
**DPA-side verbs (`doca_dpa_dev_verbs_*`) are used ONLY inside the DPA kernel.**
- They are linked by `dpacc` and are NOT available to the host-side application.
- **Rule**: Never attempt to call DPA-side verbs from the host; use the `doca-dpa` host API for management.

### 3. The "Progress the PE" Mandate
**If a kernel is launched but no completion surfaces, the first check is always the host-side PE progress.**
- Ensure `doca_pe_progress()` is being called in the main loop to drain completions.

## Diagnostic Ladder (Overlay)

| Layer | DPA Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Transport)** | `UNAVAILABLE` or `DOCA_ERROR_DRIVER`. | Check `doca_caps`; verify the BlueField is in the correct mode. |
| **Layer 2 (Program)** | `DOCA_ERROR_INVALID_VALUE` at launch. | Verify the host-DPA signature match; check launch argument types/sizes. |
| **Layer 3 (Runtime)** | No completion events after launch. | Confirm `doca_pe_progress()` is active; check DPA-side logs/stats. |
| **Layer 4 (IO Failure)** | `DOCA_ERROR_IO_FAILED` in the CQE. | Inspect the CQE error field; cross-reference with `doca-verbs` IO_FAILED logic. |
| **Layer 5 (DPA-Internal)** | Kernel stuck or hung. | Use DPA-side developer tools (Debugger, State Inspector) to analyze the processor state. |

## Command Appendix

### DPA-Specific Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `pkg-config --modversion doca-dpa` | Build-time library version check. | Semver matching `doca_caps --version`. |
| `which dpacc && dpacc --version` | Compiler availability and version check. | Version compatible with the DOCA release. |
| `doca_caps --list-devs` | Hardware DPA capability check. | Device explicitly advertises DPA support. |
| `ls /opt/mellanox/doca/samples/doca_dpa/` | Identify starting samples. | List of sample directories with both host and DPA source. |
| `DOCA_LOG_LEVEL=trace ./<binary>` | Host-side lifecycle trace. | Trace-level lines for every launch submit. |

## Deferred Topic Boundaries
- **Install/Setup**: Installing DOCA and `dpacc` is handled by `doca-setup`.
- **DPA Kernel Dev**: The internal logic of the kernel and `dpacc` flags are handled via the public *DOCA DPA* guides.
- **DPA-Side Tooling**: Using the DPA debugger or process inspector is handled by the *DPA Tools* umbrella in `doca-public-knowledge-map`.
