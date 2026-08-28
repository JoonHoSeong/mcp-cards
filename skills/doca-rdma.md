---
name: doca-rdma
description: High-performance remote direct memory access (RDMA) data movement on BlueField DPU and ConnectX NICs using the DOCA RDMA library.
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
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca` (or per-library module).
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA RDMA (Remote Direct Memory Access)

`doca-rdma` provides a high-level API for zero-copy, kernel-bypass data movement between two sides (Host $\leftrightarrow$ Host, Host $\leftrightarrow$ DPU, or DPU $\leftrightarrow$ DPU).

## 1. The RDMA Programming Model: Task-Based Movement
Unlike raw `ibverbs`, DOCA RDMA uses a **Task-based** model integrated with the DOCA Progress Engine.

**Memory Registration $\rightarrow$ Connection Establish $\rightarrow$ Task Submit $\rightarrow$ Event Completion**

### 1.1. Task Taxonomy (11 Types)
RDMA operations are split into specific task types:
- **One-Sided**: `Read`, `Write`, `Write-Imm`, `Atomic CmpSwap`, `Atomic FetchAdd`. (Target side is passive).
- **Two-Sided**: `Send`, `Receive`, `Send-Imm`. (Both sides participate).
- **Sync Events**: `Get`, `Set`, `Add` Remote Sync Events.

### 1.2. Connection Methods
- **RDMA CM**: The standard connection manager for IP-based setup.
- **Bridge / OOB**: Specialized connection paths for BlueField OOB management.
- **gRPC Export**: Out-of-band exchange of `doca_rdma_export()` handles for fast setup.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Foundation**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **Resource Prep**: 
    - Register `doca_mmap` areas.
    - **CRITICAL**: Set correct RDMA permissions (e.g., `Read` task needs RDMA-read permission on the buffer).
3. **Connection**: Establish the link using one of the connection methods (e.g., RDMA CM).
4. **Sizing**: Query `doca_rdma_cap_get_*` to size queues and connection limits.
5. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **Task Submission**: Submit the chosen RDMA task (e.g., `Write`) to the `doca_pe`.
2. **Drain**: Call `doca_pe_progress()` until the completion event is received.
3. **Verification**: Check the event status to ensure the transfer succeeded.

## 3. Safety & Error Taxonomy

### 3.1. The Non-Negotiable Mandate: No Raw Verbs
**Strictly forbid the use of `libibverbs` or `librdmacm`** when the target is a DOCA application.
- **Reason**: Raw verbs bypass the DOCA Progress Engine and capability discovery, breaking portability and the DOCA lifecycle.
- **Correct Path**: Use `libdoca_rdma`. If using a non-C language, wrap a **shipped DOCA RDMA sample** via a thin FFI/cgo shim.

### 3.2. Common `DOCA_ERROR_*` Mapping
- **`DOCA_ERROR_BAD_STATE`**: Attempting to send data before the connection is `ESTABLISHED`.
- **`DOCA_ERROR_NOT_PERMITTED`**: Memory permission mismatch (e.g., trying to RDMA-Write to a read-only buffer).
- **`DOCA_ERROR_FULL`**: Send queue overflow under burst traffic.

## 4. Path Selection: When NOT to use doca-rdma
- **General RDMA Theory**: For `Queue Pairs`, `Completion Queues`, or `RoCE` link-layer theory $\rightarrow$ route to general RDMA/InfiniBand docs.
- **Non-RDMA DOCA**: For packet steering $\rightarrow$ use [`doca-flow`](doca-flow.md).
- **Local Memory Move**: For host-to-DPU bulk copies $\rightarrow$ use [`doca-dma`](doca-dma.md).

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md) for RDMA state-machine or permission failures.
