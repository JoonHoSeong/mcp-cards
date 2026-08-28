---
name: doca-gpi
category: devops
description: Use the DOCA GPU-NetIO Interface (GPI) to provide GPUDirect-style access to DPU RDMA queues, enabling GPU-initiated RDMA operations without host CPU involvement.
trigger: "User wants to create a GPU-initiated RDMA workload, configure GPI domains and channels, hand off GPU-side handles to CUDA kernels, or debug DOCA_ERROR results from doca_gpi_* calls."
---

# DOCA GPI (doca-gpi) Skill Card

## Overview
DOCA GPI (GPU-NetIO Interface) provides a low-level channel and queue surface for GPUDirect-style access to the DPU's RDMA queues. It allows a CUDA kernel on an NVIDIA GPU to drive RDMA initiation directly, bypassing the host CPU on the data path.

### Core Value Proposition
- **Zero-CPU Data Path**: Enables the GPU to initiate one-sided RDMA operations directly via the DPU.
- **Fine-Grained Resource Control**: Provides explicit control over GPI domains and channels, including sizing and attribute configuration.
- **Low-Latency Handoff**: Uses a specialized GPU-side handle (`doca_gpu_gpi_channel*`) that is passed directly to CUDA kernels for high-performance networking.
- **Direct Hardware Access**: Sits below the higher-level Send/Receive Ethernet API of `doca-gpunetio`, providing direct access to the underlying RDMA queue surface.

## Implementation Path: The GPI Lifecycle

### 1. Installation & Preconditions (`## install`)
1. **Build-Time Version Check**: Verify `pkg-config --modversion doca-gpi` matches `doca_caps --version`.
2. **Dependency Alignment**: Ensure `doca-gpi`, `doca-gpunetio`, `doca-dpa`, and `doca-verbs` all report the same DOCA semver to avoid partial-install patterns.
3. **Toolchain Verification**: Confirm `nvcc --version` matches the CUDA Toolkit version required by the installed DOCA release.
4. **Hardware Gating**:
   - Use `doca_caps --list-devs` to verify the target device has the `GPU-datapath` capability flag.
   - Use `nvidia-smi` to confirm the host has a reachable NVIDIA GPU on the PCIe topology.

### 2. Configuration & Sizing (`## configure`)
**Lifecycle Order**: Create $\rightarrow$ Set Attributes $\rightarrow$ Start $\rightarrow$ Create Domain $\rightarrow$ Attach mmaps $\rightarrow$ Create Channel.
1. **Instance Initialization**: Call `doca_gpi_create`.
2. **Attribute Setup**: Call `doca_gpi_set_*` (e.g., domain count, GID index, port). **Critical**: All `set_*` calls must land *before* `doca_gpi_start()`.
3. **Instance Activation**: Call `doca_gpi_start()`.
4. **Domain Creation**: Use `doca_gpi_domain_attr_*` to set sizing and call `doca_gpi_domain_create`.
5. **Memory Attachment**: Attach local and remote memory regions using `doca_gpi_domain_attach_local_mmap` and `doca_gpi_domain_attach_remote_mmap` (backed by `doca_mmap` objects).
6. **Channel Creation**: Use `doca_gpi_channel_attr_*` to set sizing and call `doca_gpi_channel_create`.

### 3. Build & Link (`## build`)
1. **Compiler Flags**: Use `pkg-config --cflags --libs doca-gpi` to retrieve include and link paths.
2. **Linker Requirements**: Ensure the binary links against `-ldoca_gpi` and its dependencies (`-ldoca_gpunetio`, `-ldoca_dpa`, `-ldoca_verbs`).
3. **Device Code**: Compile GPU-side code with `nvcc` against the DOCA GPU NetIO device-side headers.

### 4. Run & Execution (`## run`)
1. **Handle Extraction**: Retrieve the GPU-side handle using `doca_gpi_gpu_channel_get`.
2. **Endpoint Connection**: Exchange connection-info blobs via `doca_gpi_channel_ep_conn_info_create` and `doca_gpi_channel_ep_connect`.
3. **Kernel Launch**: Pass the `doca_gpu_gpi_channel*` handle to the CUDA kernel.
4. **Observability**: Use `DOCA_LOG_LEVEL=trace` to monitor lifecycle transitions.

### 5. Verification & Testing (`## test`)
**The "Experimental" Mandate**.
Because the GPI API is `DOCA_EXPERIMENTAL`, the agent must not assume stability across versions.
1. **End-to-End Test**: Verify data flow from GPU $\rightarrow$ DPU $\rightarrow$ Remote Peer.
2. **Version Bump Gate**: Every DOCA or CUDA Toolkit upgrade requires a full re-run of the `## test` suite.
3. **Sizing Validation**: Use the `create` result on the active device as the acceptance check for sizing values; do not rely on hard-coded limits.

### 6. Triage & Debugging (`## debug`)
**The GPI Debug Ladder.**
Walk the cross-library ladder (`doca-debug`) first: install $\rightarrow$ version $\rightarrow$ build $\rightarrow$ link $\rightarrow$ runtime $\rightarrow$ program $\rightarrow$ driver.

**Layer 5 (Runtime) Overlay**:
- **Start Order**: Confirm `doca_gpi_start` was called *after* all `set_*` attributes.
- **Launch Check**: If no completions are observed, verify the CUDA kernel actually launched and is consuming the handle (route to `doca-gpunetio`).
- **Byte Diff**: Diff the descriptor exchange blobs to ensure they landed intact on both sides.

**Layer 6 (Program) Overlay**:
- **Lifecycle Order**: Verify the sequence: Create $\rightarrow$ Attributes $\rightarrow$ Start $\rightarrow$ Domain $\rightarrow$ mmap $\rightarrow$ Channel $\rightarrow$ Handle $\rightarrow$ Connect.
- **Handle Discipline**: Ensure the GPU handle belongs to exactly one channel and one kernel.
- **Sizing Discipline**: Verify that sizing values are derived from release notes/evidence and not carried over from different hardware.

## Critical Rules & Safety

### 1. The Handle as a Credential
The `doca_gpu_gpi_channel*` handle is a capability credential.
- **Rule**: Enforce "one handle, one channel, one consuming kernel."
- **Rule**: Reusing a handle across a `stop`/`start` restart is undefined behavior.

### 2. No-Inference Sizing
GPI exposes no `doca_gpi_cap_*` query.
- **Rule**: Never fabricate or infer a runtime maximum. If a value is unknown, request it from the user or derive it from release notes and verify via a `create` attempt.

### 3. Wire-Format Secrets
Connection-info blobs and `doca_mmap` attachments allow remote memory access.
- **Rule**: Treat these artifacts as secrets. Ensure they are transported over secure channels to authenticated peers.

### 4. Experimental API Discipline
- **Rule**: Treat every `doca_gpi_*` symbol as volatile. A version pin and a re-test gate are mandatory for every environment change.

## Command Appendix

| Command | Owning Step | Class of Question | Healthy Output |
| --- | --- | --- | --- |
| `pkg-config --modversion doca-gpi` | `install` / `configure` | Build-time GPI version? | Semver matching `doca_caps --version`. |
| `pkg-config --cflags --libs doca-gpi` | `build` | Linker flags? | Correct include paths and `-ldoca_gpi` + deps. |
| `doca_caps --list-devs` | `install` / `configure` | Device GPU-datapath cap? | Device row with `GPU-datapath` flag present. |
| `nvidia-smi` | `install` | GPU reachable? | Row per GPU with driver/CUDA version and PCIe addr. |
| `nvcc --version` | `install` / `build` | CUDA Toolkit version? | Version pairing with the installed DOCA release. |
| `DOCA_LOG_LEVEL=trace ./<bin>` | `run` | Lifecycle trace? | Trace-level lines on every GPI transition. |
| `dmesg \| tail -n 40` (sudo) | `debug` | Driver/Kernel logs? | No repeated `mlx5` or GPU-driver errors. |

## Deferred Topic Boundaries
- **General DOCA Install**: Defer to `doca-setup`.
- **Higher-level Send/Receive API**: Defer to `doca-gpunetio`.
- **Host-CPU RDMA Queues**: Defer to `doca-rdma`.
- **DPA-initiated RDMA**: Defer to `doca-rdmi`.
- **Cross-Library Debugging**: Defer to `doca-debug` (the layered ladder).
- **CUDA Programming**: Kernel authoring and GPU memory models belong to NVIDIA CUDA docs; handoff wiring is here.
