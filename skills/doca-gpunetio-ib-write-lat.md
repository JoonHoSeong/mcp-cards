---
name: doca-gpunetio-ib-write-lat
category: devops
description: Use the `gpunetio_ib_write_lat` tool to measure the RDMA WRITE latency distribution between a GPU and a remote peer.
trigger: "User wants to measure GPU-initiated RDMA WRITE latency, analyze p99/p99.9 tail latency, or baseline GPUNetIO real-time performance against a specific GPU-NIC pairing."
---

# DOCA GPUNetIO IB Write Latency (doca-gpunetio-ib-write-lat) Skill Card

## Overview
`gpunetio_ib_write_lat` is a specialized measurement tool used to analyze the latency distribution of GPU-initiated RDMA WRITE operations. Unlike bandwidth tools, this focuses on the "time-to-completion" for a single operation, providing critical data for real-time and control-loop workloads.

### Core Value Proposition
- **Tail Latency Analysis**: Provides detailed statistics (median, p99, p99.9) to identify jitter and outlier events.
- **Real-Time Baseline**: Establishes the latency floor for GPU-driven networking, enabling the design of deadline-bound applications.
- **Pairing Validation**: Verifies that the GPU-NIC PCIe/NVLink topology minimizes latency for the target workload.
- **Regression Tracking**: Detects latency shifts across DOCA/CUDA versions or firmware updates.

## Implementation Path: The Measurement Lifecycle

### 1. Installation & Configuration (`## configure`)
1. **Build-Time Alignment**: Verify `pkg-config --modversion doca-gpunetio` matches the DOCA release and `nvcc --version` aligns with the Compatibility Policy.
2. **Binary Discovery**: Locate binaries in `/opt/mellanox/doca/tools/gpunetio_ib_write_lat/{client,server}/`. Do not search in `/samples/`.
3. **Environment Check**: Confirm `nvidia_peermem` is loaded on both ends to enable GPUDirect RDMA.
4. **Topology Audit**: Document the GPU and NIC models and their PCIe addresses on both hosts to ensure the GPU-NIC pairing is recorded.

### 2. Execution Flow (`## run`)
**Mandatory Order: Server First $\rightarrow$ Client Second.**
1. **Server Launch**: Run the `server` binary on the remote host.
   - Monitor the "pre-run echo" to confirm the IB device, GPU PCIe address, and GID index.
2. **Client Launch**: Run the `client` binary on the local host.
   - Flags: `-c <server-ip>`, `-d <ib-device>`, `--gpu <gpu-ordinal>`, and `--gid-index <index>` (if non-default).
3. **Smoke Run**: Execute a single iteration to verify the path is healthy. Verify that the reported `t_cuda` and `t_full_iter` are in a defensible order of magnitude.
4. **Statistical Run**: Run the tool with sufficient iterations to populate the latency tail (e.g., $10^5$ for p99, $10^6$ for p99.9).

### 3. Soundness Verification (`## test`)
**The Eval-Loop Overlay.**
A run that completes is not a "sound" measurement. The agent must iterate until the distribution is defensible:

| Observation | Hypothesis | Next Action |
| --- | --- | --- |
| Median $\gg$ NIC Latency Floor | CUDA-side timer artifact | Cross-check `t_cuda` against host-side `t_full_iter`; re-verify warm-up. |
| p99 $\gg$ Median | Real Tail Latency / Jitter | Quote p99 as the primary answer for real-time workloads; capture the outlier count. |
| Low Iteration Count | Kernel-side timeout | Verify the source constant for `NUM_ITER`; route to a bespoke benchmark if more samples are needed. |
| Distribution Shift | Hardware/Driver Delta | Capture the tuple on both hosts; route to `doca-version` or `doca-setup`. |

### 4. Triage & Debugging (`## debug`)
**The Latency-Debug Ladder.**
1. **Config-Syntax**: Verify flags against the binary's `--help`.
2. **Build/Link**: Check for missing `doca-gpunetio.pc` or `nvcc` mismatches.
3. **Pairing**: Re-verify the GPU-NIC PCIe/NVLink topology.
4. **Lifecycle**: Check for `nvidia_peermem` loading or incorrect buffer registration order.
5. **Connection**: Verify GID index match and RDMA WRITE permissions.
6. **Soundness**: Walk the Eval-Loop to confirm the reported statistic is derivable from the data.

## Critical Rules & Safety

### 1. The Tuple Mandate
**Distributions without the full tuple are unfalsifiable.**
Every captured baseline must include:
- **Software**: `pkg-config --modversion doca-gpunetio` and `nvcc --version`.
- **Hardware**: GPU model/PCIe addr and NIC model/PCIe addr on both hosts.
- **Topology**: NUMA node and PCIe root complex pairing.
- **Config**: Exact client and server command lines used.
- **Statistic**: The specific metric used (median, p99, p99.9, or jitter).

### 2. The Statistic Rule
**Never quote a single number without naming the statistic.**
- **Rule**: Every reported latency must be explicitly labeled (e.g., *"p99 latency = 1.2 $\mu$s"*). Quoting a number without the statistic is a cross-tool comparison failure.

### 3. No-Inference Rule
- **Rule**: Do not invent flags or assume `NUM_ITER` can be changed at runtime. Always check the binary's `--help` and source.

### 4. Redaction Requirement
- **Rule**: Before persisting or sharing logs, redact GPU-side handles, memory-mmap exports, and OOB connection descriptors.

## Command Appendix

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `/opt/.../server/gpunetio_ib_write_lat` | Start remote listener. | Echo of IB device, GPU PCIe address, and "Server listening". |
| `/opt/.../client/gpunetio_ib_write_lat -c <IP> -d <dev> --gpu <id>` | Start local sender. | Distribution of latency (median, p99, etc.) and `t_cuda`. |
| `pkg-config --modversion doca-gpunetio` | Verify library version. | Semver matching the DOCA release. |
| `lsmod \| grep nvidia_peermem` | Verify GPUDirect RDMA. | Module is listed as loaded. |
| `nvidia-smi dmon` | Check GPU SM occupancy. | Low SM utilization during the benchmark run. |
| `ibstat` | Check link rate. | Link speed matching the hardware spec. |

## Deferred Topic Boundaries
- **GPUNetIO Programming**: Authoring bespoke latency benchmarks $\rightarrow$ `doca-gpunetio`.
- **Write Bandwidth**: Measuring throughput instead of latency $\rightarrow$ `doca-gpunetio-ib-write-bw`.
- **CPU-initiated Latency**: Using standard `perftest` $\rightarrow$ `doca-public-knowledge-map`.
- **GPI Programming**: Measuring the same operation via the GPI surface $\rightarrow$ `doca-gpi`.
- **Hardware Changes**: Firmware burns or IOMMU changes $\rightarrow$ `doca-hardware-safety`.
- **Installation**: DOCA/CUDA setup $\rightarrow$ `doca-setup`.
