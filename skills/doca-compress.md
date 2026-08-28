---
name: doca-compress
description: Offload bulk DEFLATE compression and decompression to BlueField DPU/ConnectX hardware to reduce CPU overhead and memory bandwidth.
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
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-compress`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA Compress (DEFLATE / LZ4)

`doca-compress` offloads bulk data compression and decompression to the hardware, specifically supporting the DEFLATE algorithm (used in gzip/zlib) and LZ4 decompression.

## 1. The Compression Model: Task-Based Offload
DOCA Compress operates as a task-based system where you define a source buffer, a destination buffer, and the operation type.

**Source `doca_buf` $\rightarrow$ [Hardware Accelerator] $\rightarrow$ Destination `doca_buf`**

### 1.1. Supported Algorithms
- **DEFLATE (Compress & Decompress)**: The canonical fit for bulk data.
- **LZ4 (Decompress only)**: High-speed decompression for pre-compressed streams.
- **Note**: LZ4 encoding is NOT supported in hardware; use a CPU LZ4 library.

### 1.2. The Size-Threshold Rule (When to Offload)
Compression offload is not always faster due to the DMA setup cost.
- **Bulk Data ($\ge$ a few KiB)**: Hardware offload is the canonical fit.
- **Tiny Data ($<$ a few KiB)**: CPU-based `zlib` or `zstd` is typically faster.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Common Setup**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **Context Creation**: Create a `doca_compress` context.
3. **Algorithm Selection**: Choose between `compress-deflate`, `decompress-deflate`, `decompress-lz4-stream`, or `decompress-lz4-block`.
4. **Capability Check**: 
    - Call `doca_compress_cap_task_*_is_supported` to verify hardware support for the chosen task.
    - Call `doca_compress_cap_task_*_get_max_buf_size` to size the source buffer.
5. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **Buffer Prep**: Register `doca_buf` for source and destination.
2. **Permissions**:
    - Source: `DOCA_ACCESS_FLAG_LOCAL_READ_ONLY`.
    - Destination: `DOCA_ACCESS_FLAG_LOCAL_READ_WRITE`.
3. **Submit**: Submit the compression/decompression task to the `doca_pe`.
4. **Drain**: Call `doca_pe_progress()` until the completion event is received.

## 3. Safety & Error Taxonomy

### 3.1. Common Error Patterns
- **`DOCA_ERROR_INVALID_VALUE`**: Typically caused by:
    - Attempting to use an unsupported algorithm (e.g., LZ4 encode).
    - Source buffer size exceeding the `max_buf_size` returned by the capability query.
    - Destination buffer too small to hold the decompressed output.
- **`DOCA_ERROR_NOT_PERMITTED`**: Memory permission mismatch on the `doca_buf`.

### 3.2. Validation Strategy
Always perform a **Round-Trip Smoke Test**:
- **Compress $\rightarrow$ Decompress $\rightarrow$ Compare**: Compress a known buffer, decompress it back, and verify that the result matches the original input exactly.

## 4. Path Selection: When NOT to use doca-compress
- **Other Algorithms**: For Zstd, Brotli, or Snappy $\rightarrow$ use a CPU library.
- **Tiny Payloads**: Use CPU `zlib`/`zstd` for small inputs to avoid DMA overhead.
- **No Compression**: For pure memory moves $\rightarrow$ use [`doca-dma`](doca-dma.md).

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md) for hardware-level compression hangs.
