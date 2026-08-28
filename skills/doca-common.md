---
name: doca-common
description: Foundation primitives for all DOCA apps: ctx lifecycle, dev discovery, zero-copy buf wiring, and the two-tier logging model.
license: Apache-2.0
metadata:
  kind: library
  layer: foundation
  dependencies:
    - doca-setup
  routes_to:
    - doca-programming-guide
    - doca-debug
    - doca-hardware-safety
compatibility: >
  Requires DOCA SDK installed at /opt/mellanox/doca on Linux with a BlueField DPU or ConnectX NIC. 
  Authority: `pkg-config --modversion doca-common`.
---

# DOCA Common (Foundation Layer)

`doca-common` is the absolute base layer for every DOCA application. No higher-level library (Flow, RDMA, DMA, etc.) can function without the primitives defined here. It provides the universal "glue" between the OS, the BlueField hardware, and the DOCA API.

## 1. The Universal Foundation Model
Every DOCA application, regardless of its specific purpose, must follow this foundation walk:

**`doca_devinfo` (Discovery) $\rightarrow$ `doca_dev` (Handle) $\rightarrow$ `doca_ctx` (Context) $\rightarrow$ `doca_pe` (Progress Engine) $\rightarrow$ `doca_ctx_start` (Execution)**

### 1.1. Device Discovery & Gating
Do not assume a feature is supported based on documentation alone. Always gate on the active `doca_devinfo`.
- **Discovery**: `doca_devinfo_create_list` $\rightarrow$ iterate to find target DPU/NIC.
- **Gating**: Use the `doca_<library>_cap_*` family of functions against the `doca_devinfo` handle. If a capability query returns `false`, the app must fail-fast or fall back to a CPU implementation.

### 1.2. Zero-Copy I/O Model (The Buffer Chain)
DOCA achieves high performance by eliminating CPU copies. This is managed through a strict hierarchy:
**`doca_mmap` (Region) $\rightarrow$ `doca_buf_inventory` (Slicer) $\rightarrow$ `doca_buf` (Segment)**

- **`doca_mmap`**: Registers a large contiguous memory region with the hardware.
- **`doca_buf_inventory`**: Manages the division of a mmap region into smaller, fixed-size buffers.
- **`doca_buf`**: The actual handle passed to DMA/RDMA/Flow tasks.
- **Cross-Library Sharing**: A `doca_buf` created in `doca-common` can be passed directly to `doca-dma` for transfer, then to `doca-rdma` for network transmission, without ever leaving the hardware-accessible memory.

### 1.3. The Progress Engine (doca_pe)
DOCA is asynchronous. Submitting a task does not mean it is complete.
- **Mechanism**: The Progress Engine (`doca_pe`) is the universal "drain" for completions.
- **The Run-Loop**: You must call `doca_pe_progress()` in a tight loop (or a dedicated thread) to trigger the completion callbacks of all connected contexts.
- **Common Failure**: "Task submitted but nothing happens" $\rightarrow$ check if `doca_pe_connect_ctx` was called and if the `doca_pe_progress` loop is actually running.

### 1.4. Two-Tier Logging Model
DOCA logs are split into two distinct control planes:
1. **SDK-Level (`--sdk-log-level`)**: Controls the internal NVIDIA SDK logs. Set via environment variable or CLI flag.
2. **App-Level (`doca_log_*`)**: Controls logs emitted by the user's application code.
- **Critical Path**: If `DOCA_LOG_DBG` lines are missing, ensure the app-side registry is configured correctly AND that the SDK-level is not filtering them out.

## 2. Operational Workflows

### 2.1. Initialize Foundation (`configure`)
1. **Device Discovery**: Create `doca_devinfo` list.
2. **Context Setup**: Create `doca_ctx` (and per-library contexts on top of it).
3. **PE Wiring**: Create `doca_pe` $\rightarrow$ `doca_pe_connect_ctx(pe, ctx)`.
4. **Execution Start**: `doca_ctx_start(ctx)`.

### 2.2. Zero-Copy Buffer Setup (`use`)
1. **Mmap**: `doca_mmap_create` (Allocate/Register memory).
2. **Inventory**: `doca_buf_inventory_create` (Define buffer size/count).
3. **Allocation**: `doca_buf_inventory_alloc` $\rightarrow$ returns `doca_buf`.

## 3. Error Taxonomy & Debugging

### 3.1. Common `DOCA_ERROR_*` Mapping
- **`DOCA_ERROR_BAD_STATE`**: Typically occurs during `doca_ctx_start` if the PE is not connected or the device is in an incompatible state.
- **`DOCA_ERROR_NOT_PERMITTED`**: Memory permissions mismatch (e.g., attempting to write to a read-only `doca_mmap`).
- **`DOCA_ERROR_INVALID_VALUE`**: Buffer size or alignment does not match the device's capability query (`doca_*_cap_*`).

### 3.2. Debug Ladder
1. **Log Check**: Verify `--sdk-log-level DEBUG`.
2. **Capability Check**: Re-run `doca_*_cap_*` queries to ensure the hardware actually supports the requested feature.
3. **Lifecycle Check**: Ensure `ctx_start` $\rightarrow$ `pe_progress` $\rightarrow$ `ctx_stop` sequence is strictly followed.
4. **Escalation**: If the error persists, route to [`doca-debug`](doca-debug.md) for driver/firmware level analysis.

## 4. Routing & Dependencies
- **Before this**: Use [`doca-setup`](doca-setup.md) to install the SDK and verify the DPU.
- **After this**: Use [`doca-programming-guide`](doca-programming-guide.md) for build patterns (Meson/pkg-config).
- **For Features**: Load this skill alongside specific libraries (e.g., [`doca-dma`](doca-dma.md), [`doca-flow`](doca-flow.md)).
