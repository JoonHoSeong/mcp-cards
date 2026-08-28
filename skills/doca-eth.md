---
name: doca-eth
description: High-performance Ethernet packet I/O on BlueField DPU and ConnectX NICs using the DOCA Ethernet library for line-rate RX/TX.
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
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-eth`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA Ethernet (Packet I/O)

`doca-eth` provides the interface for high-throughput, line-rate packet transmission and reception on NVIDIA NICs/DPUs. It is the "Data Plane" surface, whereas `doca-flow` is the "Steering Plane" surface.

## 1. The Ethernet Queue Model: RX and TX Split
DOCA Ethernet separates the receive and transmit paths into distinct objects to allow independent tuning.

### 1.1. RX Queue Types (`doca_eth_rxq_type`)
The choice of RX type depends on the data shape and memory management strategy:
- **`_REGULAR`**: Standard queue for general-purpose packet reception.
- **`_CYCLIC`**: Optimized for fixed-size frames in a pre-allocated buffer ring.
- **`_MANAGED_MEMPOOL` / `_SHARED_MEMPOOL`**: Used for complex memory sharing and zero-copy handoffs between multiple consumers.

### 1.2. TX Submission Model
TX operations are task-based. You do not "write" to a socket; you submit a `doca_eth_txq_task_send` (or `_lso_send` for Large Send Offload) to the Progress Engine.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Foundation**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **RX Queue Setup**:
    - Select `rxq_type` based on the data shape.
    - Query `doca_eth_rxq_cap_is_type_supported` to ensure hardware compatibility.
    - Configure burst size and buffer lengths.
3. **TX Queue Setup**:
    - Configure scatter-gather length and L3/L4 checksum offload capabilities.
4. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **RX Loop**: 
    - Poll the `doca_pe` for RX completion events.
    - Access packet data via the associated `doca_buf`.
2. **TX Send**:
    - Allocate a `doca_buf` for the packet.
    - Submit the send-task to the `doca_pe`.
    - Wait for the completion event to recycle the buffer.

## 3. Safety & Error Taxonomy

### 3.1. The Steering Dependency (Critical)
**An empty RX queue is almost never an Ethernet bug; it is a Flow bug.**
`doca-eth` provides the *bucket* (the queue), but `doca-flow` provides the *pipe* (the steering rule). If no packets arrive:
- Check if a `doca-flow` rule is actively steering traffic to the specific RX queue ID.
- For first-run testing, use kernel-side promiscuous mode (via `doca-setup`) to force all traffic into the queue.

### 3.2. Common `DOCA_ERROR_*` Mapping
- **`DOCA_ERROR_AGAIN`**: Returned by `doca_task_submit` when the TX queue is full. Implement a retry-backoff or use `doca_pe_progress()` to clear completions.
- **`DOCA_ERROR_INVALID_VALUE`**: Incorrect burst size or unsupported RX type for the given hardware.
- **`DOCA_ERROR_NOT_PERMITTED`**: Missing `sudo` or `mlnx` group permissions to open the `doca_dev` against a physical port.

## 4. Path Selection: When NOT to use doca-eth
- **Packet Steering**: To define *which* packets go to *which* queue $\rightarrow$ use [`doca-flow`](doca-flow.md).
- **Control Messaging**: For host $\leftrightarrow$ DPU command-and-control $\rightarrow$ use [`doca-comch`](doca-comch.md).
- **Bulk RDMA**: For zero-copy memory movement $\rightarrow$ use [`doca-rdma`](doca-rdma.md).

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Steering**: [`doca-flow`](doca-flow.md) (Mandatory for production traffic).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md).
