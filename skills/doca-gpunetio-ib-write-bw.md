---
name: doca-gpunetio-ib-write-bw
category: devops
description: Use the `gpunetio_ib_write_bw` tool to measure the sustained RDMA WRITE throughput between a GPU and a remote peer.
trigger: "User wants to measure GPU-initiated RDMA WRITE bandwidth, conduct throughput sweeps across message sizes, or baseline GPUNetIO performance against a specific GPU-NIC pairing."
---

# DOCA GPUNetIO IB Write BW (doca-gpunetio-ib-write-bw) Skill Card

## Overview
`gpunetio_ib_write_bw` is a dedicated performance measurement tool used to determine the sustained RDMA WRITE throughput for GPU-initiated networking. It evaluates the entire path from the GPU's persistent kernel, through the DPU's RDMA queues, across the network, and into the remote peer's memory.

### Core Value Proposition
- **Path-Specific Benchmarking**: Isolates the GPU-driven RDMA WRITE path to identify performance bottlenecks.
- **Binding Constraint Identification**: Distinguishes between GPU compute occupancy, NIC issue rate, and physical link saturation.
- **Regression Detection**: Provides a verifiable baseline for detecting performance shifts across DOCA/CUDA versions or firmware updates.
- **Pairing Validation**: Verifies that the GPU-NIC PCIe/NVLink topology is optimal for the target workload.

## Implementation Path: The Measurement Lifecycle

### 1. Installation & Configuration (`## configure`)
1. **Build-Time Alignment**: Verify `pkg-config --modversion doca-gpunetio` matches the installed DOCA release and `nvcc --version` aligns with the Compatibility Policy.
2. **Binary Discovery**: Locate the binaries in `/opt/mellanox/doca/tools/gpunetio_ib_write_bw/{client,server}/`. Do not search in `/samples/`.
3. **Environment Check**: Confirm `nvidia_peermem` is loaded on both ends to enable GPUDirect RDMA.
4. **Topology Audit**: Document the GPU model, PCIe address, and NIC model/address on both hosts to ensure the GPU-NIC pairing is known.

### 2. Execution Flow (`## run`)
**Mandatory Order: Server First $\rightarrow$ Client Second.**
1. **Server Launch**: Run the `server` binary on the remote host.
   - Monitor the "pre-run echo" to confirm the IB device and GID index.
   - The server uses an OOB TCP socket for connection; it has no `--gpu` argument.
2. **Client Launch**: Run the `client` binary on the local host.
   - Flags: `-c <server-ip>`, `-d <ib-device>`, `--gpu <gpu-ordinal>`, and `--gid-index <index>` (if non-default).
3. **Smoke Run**: Execute a single run with a standard message size to verify the path is healthy.
4. **Bulk/Sweep Run**: Perform sweeps across message sizes and iteration counts using the flags found in `--help`.

### 3. Soundness Verification (`## test`)
**The Eval-Loop Overlay.**
A completed run is not necessarily a "sound" measurement. The agent must iterate until the binding constraint is identified:

| Observation | Hypothesis | Next Action |
| --- | --- | --- |
| BW $\ll$ Link Capacity | Pairing or Topology issue | Re-verify GPU-NIC PCIe pairing; check NUMA pinning. |
| BW varies with Msg Size but not Concurrency | NIC-issue-rate bound | Compare result against the NIC's documented transport submission rate. |
| BW varies with Concurrency but not Msg Size | GPU-compute occupancy bound | Re-examine the persistent-kernel pattern in `doca-gpunetio`. |
| BW is stable but below capacity | Link saturation / Peer bottleneck | Check `ibstat` on both ends; verify peer CPU/Memory bandwidth. |

### 4. Triage & Debugging (`## debug`)
**The BW-Debug Ladder.**
1. **Config-Syntax**: Verify flags against the binary's `--help`.
2. **Build/Link**: Check for missing `doca-gpunetio.pc` or `nvcc` mismatches.
3. **Pairing**: Re-verify the GPU-NIC PCIe/NVLink topology.
4. **Lifecycle**: Check if `nvidia_peermem` is loaded or if buffer registration happened after `doca_ctx_start()`.
5. **Connection**: Verify GID index match and RDMA WRITE permissions.
6. **Soundness**: Walk the Eval-Loop to confirm the binding constraint.
7. **Version**: Cross-check DOCA + CUDA versions on both build and run hosts.

## Critical Rules & Safety

### 1. The Tuple Mandate
**Measurements without the full tuple are unfalsifiable.**
Every captured baseline must include:
- **Software**: `pkg-config --modversion doca-gpunetio` and `nvcc --version`.
- **Hardware**: GPU model/PCIe addr and NIC model/PCIe addr on both hosts.
- **Topology**: NUMA node and PCIe root complex pairing.
- **Config**: Exact client and server command lines used.
- **Result**: Full stdout from both halves (redacted for secrets).

### 2. The Binding Constraint Rule
**Never quote a number in isolation.**
- **Rule**: Every reported throughput must be accompanied by its identified binding constraint (e.g., *"120 Gbit/s, NIC-issue-rate-bound"*).

### 3. No-Inference Rule
- **Rule**: Do not invent sweep flags or default values. Always use the binary's `--help`.

### 4. Redaction Requirement
- **Rule**: Before persisting or sharing logs, redact GPU-side handles, memory-mmap exports, and OOB connection descriptors.

## Command Appendix

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `/opt/.../server/gpunetio_ib_write_bw` | Start remote listener. | Echo of IB device, GID index, and "Server listening". |
| `/opt/.../client/gpunetio_ib_write_bw -c <IP> -d <dev> --gpu <id>` | Start local sender. | Sustained throughput value (e.g., `BW = 150.2 Gbps`). |
| `pkg-config --modversion doca-gpunetio` | Verify library version. | Semver matching the DOCA release. |
| `lsmod \| grep nvidia_peermem` | Verify GPUDirect RDMA. | Module is listed as loaded. |
| `nvidia-smi dmon` | Check GPU SM occupancy. | High SM utilization during the benchmark run. |
| `ibstat` | Check link rate. | Link speed matching the hardware spec (e.g., 200Gbps). |

## Deferred Topic Boundaries
- **GPUNetIO Programming**: Authoring bespoke benchmarks $\rightarrow$ `doca-gpunetio`.
- **Write Latency**: Measuring latency instead of BW $\rightarrow$ `doca-gpunetio-ib-write-lat`.
- **CPU-initiated BW**: Using standard `perftest` $\rightarrow$ `doca-public-knowledge-map`.
- **Hardware Changes**: Firmware burns or IOMMU changes $\rightarrow$ `doca-hardware-safety`.
- **Installation**: DOCA/CUDA setup $\rightarrow$ `doca-setup`.
