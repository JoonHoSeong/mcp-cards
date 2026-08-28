---
name: doca-dma
description: Offload bulk memory copies between host and DPU using the BlueField DMA engine instead of the CPU.
license: Apache-2.0
metadata:
  kind: library
  layer: feature
  dependencies:
    - doca-common
  routes_to:
    - doca-programming-guide
    - doca-debug
compatibility: >
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-dma`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA DMA (Direct Memory Access)

`doca-dma` provides the ability to move large blocks of data between the host and the DPU (or between two DPU regions) using the hardware DMA engine. This offloads the copy operation from the CPU, freeing it for application logic.

## 1. The DMA Model: mmap-to-mmap
DOCA DMA does not work with raw pointers. It operates exclusively on memory regions registered via `doca-common`.

**Source `doca_buf` $\rightarrow$ `doca_dma` Task $\rightarrow$ Destination `doca_buf`**

### 1.1. The Memcpy Task (`doca_dma_task_memcpy`)
The primary operation is a bulk copy.
- **Input**: A source `doca_buf` and a destination `doca_buf`.
- **Configuration**: Use `doca_dma_task_memcpy_set_conf` to define the copy size and parameters before starting the context.
- **Execution**: Submit the task to the `doca_pe` (Progress Engine).

### 1.2. Capability Gating (The Golden Rule)
Never assume a buffer size is supported. Always query the hardware:
- **`doca_dma_cap_task_memcpy_get_max_buf_size`**: Returns the maximum size allowed for a single DMA task.
- **`doca_dma_cap_task_memcpy_get_max_buf_list_len`**: Returns the limit for scatter-gather lists.
- **Action**: If the required copy size exceeds the max buffer size, you must split the copy into multiple sequential tasks or use a scatter-gather list.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Common Setup**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **DMA Context**: Create a `doca_dma` context.
3. **Task Config**: Call `doca_dma_task_memcpy_set_conf` with the target buffer size.
4. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **Buffer Prep**: Ensure source and destination are valid `doca_buf` handles.
2. **Permission Check**: 
    - Source: Must have `DOCA_ACCESS_FLAG_LOCAL_READ_ONLY`.
    - Destination: Must have `DOCA_ACCESS_FLAG_LOCAL_READ_WRITE`.
3. **Submit**: Submit the task to the `doca_pe`.
4. **Drain**: Call `doca_pe_progress()` until the completion event for the DMA task is received.

## 3. Safety & Error Taxonomy

### 3.1. Permission Mismatches
- **`DOCA_ERROR_NOT_PERMITTED`**: The most common DMA error. Occurs when the `doca_mmap` flags do not match the task's requirements (e.g., trying to write to a read-only buffer).

### 3.2. Buffer Sizing Errors
- **`DOCA_ERROR_INVALID_VALUE`**: Usually means the buffer size exceeds the `max_buf_size` returned by the capability query.

### 3.3. Lifecycle Violations
- **`DOCA_ERROR_BAD_STATE`**: Attempting to submit a task before `doca_ctx_start` or after `doca_ctx_stop`.

## 4. Path Selection: When NOT to use DMA
Do not use `doca-dma` in the following cases:
- **Network Transfers**: If data must cross the network to another node $\rightarrow$ use [`doca-rdma`](doca-rdma.md).
- **Small Messages**: For low-latency, small-size producer/consumer flows $\rightarrow$ use [`doca-comch`](doca-comch.md).
- **Tiny Copies**: If the copy is only a few bytes, the overhead of DMA setup exceeds the cost of a CPU `memcpy`.

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md) (for Meson/pkg-config).
- **Debug**: [`doca-debug`](doca-debug.md) for hardware-level DMA hangs.
