---
name: doca-gpunetio
category: devops
description: Use DOCA GPUNetIO to enable high-performance, GPU-initiated networking by allowing CUDA kernels to directly drive the DPU's RDMA queues.
trigger: "User wants to implement GPUDirect RDMA, create a persistent GPU networking kernel, manage GPU buffer registrations, or debug DOCA_ERROR_DRIVER/NOT_SUPPORTED results from doca_gpu_* calls."
---

# DOCA GPUNetIO (doca-gpunetio) Skill Card

## Overview
DOCA GPUNetIO is the high-performance interface that enables NVIDIA GPUs to perform direct networking operations via a DPU. It allows for "persistent kernels"—CUDA kernels that stay resident on the GPU and poll/drive DPU queues—eliminating the host CPU from the data path.

### Core Value Proposition
- **GPU-Driven Data Path**: Moves the networking logic (submit/drain) into the GPU, enabling true GPUDirect RDMA.
- **Persistent Kernel Pattern**: Reduces latency by avoiding repeated kernel launches; the GPU acts as the primary network processor.
- **Direct Hardware Integration**: Tight integration between CUDA memory spaces and DOCA Ethernet/RDMA queues.
- **High-Throughput I/O**: Optimizes the transfer of large data blocks between GPU memory and the wire.

## Implementation Path: The GPUNetIO Lifecycle

### 1. Configuration & Preconditions (`## configure`)
1. **Environment Alignment**:
   - Verify `pkg-config --modversion doca-gpunetio` matches the installed DOCA release.
   - Ensure `nvcc --version` aligns with the DOCA Compatibility Policy.
2. **Hardware Gating**:
   - Confirm the DPU is in the correct mode (VNF/SF) and the GPU is reachable via PCIe.
   - Verify `nvidia_peermem` is loaded (`lsmod | grep nvidia_peermem`). If missing, `doca_gpu_create` will return `DOCA_ERROR_NOT_SUPPORTED`.
3. **Parent Queue Setup**: Ensure the underlying `doca_eth_rxq` / `doca_eth_txq` are fully configured (route to `doca-eth`).

### 2. Build & Link (`## build`)
1. **Toolchain**: Use `nvcc` for GPU code and a C/C++ compiler for host code.
2. **Linking**: Use `pkg-config --cflags --libs doca-gpunetio` to ensure correct paths to `-ldoca_gpunetio` and dependencies.
3. **CUDA Architecture**: Target the correct GPU compute capability (e.g., sm_80 for A100/H100).

### 3. Run & Execution (`## run`)
1. **Context Creation**: Initialize the GPUNetIO context via `doca_gpu_create`.
2. **Buffer Registration**: Register GPU memory buffers using `doca_buf_arr_create_*`. **Crucial**: Buffers must be registered *before* the kernel is launched.
3. **Kernel Launch**: Launch the persistent CUDA kernel, passing the GPUNetIO context and buffer handles.
4. **Data Plane Operation**: The kernel performs the submit/drain loop, interacting directly with the DPU queues.
5. **Observability**: Use `nvidia-smi -q -d UTILIZATION` to verify the GPU is actually processing the networking workload.

### 4. Verification & Testing (`## test`)
1. **Path Verification**: Verify that packets sent from the GPU are received by the peer and vice-versa.
2. **Throughput Baseline**: Measure bandwidth/latency and compare against the theoretical limit of the PCIe/Network link.
3. **Stability Test**: Run the persistent kernel for extended periods to ensure no `CUDA_ERROR_LAUNCH_FAILED` or memory leaks occur.

### 5. Triage & Debugging (`## debug`)
**The GPUNetIO Debug Ladder.**
Walk the `doca-debug` layered ladder first. For GPUNetIO-specific issues, apply the following overlays:

- **Driver Layer**: `DOCA_ERROR_DRIVER` usually indicates a CUDA driver failure. Check `dmesg` and `nvidia-smi`.
- **Support Layer**: `DOCA_ERROR_NOT_SUPPORTED` at creation almost always means `nvidia_peermem` is not loaded.

**The 5-Phase Universal Debug Loop (GPUNetIO Implementation):**
1. **Layer Identification**: Identify if the bug is RX, Lifecycle, or Driver.
2. **Triple Capture (READ-ONLY)**:
   - (a) Configure-time `doca_devinfo`, parent `doca_eth_rxq` identity, and `ethtool -S` counters.
   - (b) `DOCA_LOG_LEVEL=DEBUG` logs for the failing submit/drain.
   - (c) GPU state via `nvidia-smi -q` and `compute-sanitizer` on the kernel.
3. **Single-Variable Mutation**: Make a mutation *smaller* than the original change (e.g., halve the drain width).
4. **Re-capture and Compare**: Re-run the Triple Capture and diff against the baseline.
5. **Exit or Escalate**: Exit on a "green signal" (e.g., `rx_packets` incrementing) or escalate to `doca-debug` with the captured triple.

## Rollback Procedure
GPUNetIO is stateful. A botched sequence leaves leaked GPU buffers and hung kernels.

**The 5-Step Reversal (Mandatory Order):**
1. **Signal Kernel Stop**: Flip the host-side termination flag. Poll `cudaStreamQuery` until completion. **Do not** call `cudaDeviceSynchronize` for this drain.
2. **Unregister Buffers**: Call `doca_buf_arr_destroy` on all registered arrays in reverse-registration order.
3. **Destroy Context**: Call `doca_ctx_stop` followed by `doca_gpu_destroy`.
4. **GPU Free**: Call `cudaFree` on the underlying allocations.
5. **Re-verify Parent**: Run a smoke test on the `doca-eth` parent queue to ensure it is still intact.

## Critical Rules & Safety

### 1. The Registration Order
- **Rule**: Buffers MUST be unregistered via `doca_buf_arr_destroy` *before* the underlying memory is freed with `cudaFree`. Reversing this causes memory corruption.

### 2. The Rollback Mandate
- **Rule**: Every change-recommending answer MUST include the rollback path: *"the rollback path is the five-step reversal in ## rollback; the agent has captured the GPU allocation map and persistent-kernel stop-flag location."*

### 3. Kernel Drain Safety
- **Rule**: Never destroy a context while the GPU kernel is still active. Always use the termination flag and `cudaStreamQuery` to confirm the kernel has exited.

### 4. No-Inference Rule
- **Rule**: Do not guess CUDA version compatibility. Always check the DOCA Compatibility Policy.

## Command Appendix

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `pkg-config --modversion doca-gpunetio` | Check version match. | Semver matching `doca-common`. |
| `pkg-config --cflags --libs doca-gpunetio` | Get linker flags. | Correct `-I` paths and `-ldoca_gpunetio`. |
| `nvcc --version` | Check CUDA version. | Version matching DOCA Compatibility Policy. |
| `lsmod \| grep nvidia_peermem` | Verify GPUDirect RDMA. | Module is listed as loaded. |
| `nvidia-smi -L` | Enumerate GPUs. | List of available GPU UUIDs. |
| `nvidia-smi -q -d UTILIZATION` | Verify GPU activity. | Non-zero GPU utilization during kernel run. |
| `dmesg \| tail -n 40` | Check driver logs. | No repeated `mlx5` or `nvidia` errors. |
| `DOCA_LOG_LEVEL=trace ./<bin>` | Lifecycle tracing. | Trace lines on every transition. |
| `cat /usr/local/cuda/version.txt` | Verify CUDA install. | Matches `nvcc --version`. |

## Deferred Topic Boundaries
- **General Installation**: Defer to `doca-setup`.
- **CUDA Kernel Tuning**: Defer to NVIDIA CUDA documentation (block/thread sizing).
- **Ethernet Queue Setup**: Defer to `doca-eth`.
- **Cross-Library Debugging**: Defer to `doca-debug`.
