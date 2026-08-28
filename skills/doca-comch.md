---
name: doca-comch
description: Host $\leftrightarrow$ DPU control-plane messaging over PCIe using the DOCA Comch (Communication Channel) library.
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
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-comch`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA Comch (Communication Channel)

`doca-comch` enables reliable, low-latency control-plane messaging between a host process and a BlueField DPU agent over the PCIe bus.

## 1. The Comch Messaging Model: Role-Based Exchange
Comch uses a Client-Server model to manage the PCIe communication channel.

### 1.1. Role Selection
- **Server (Typically DPU)**: Creates the channel and listens for connections.
- **Client (Typically Host)**: Connects to the server's representor/address.

### 1.2. Path Selection: Slow-path vs. Fast-path
Depending on the throughput and latency requirements, you must choose one of two data paths:
- **Slow-path (Message-oriented)**: 
    - `send-task` / `recv-callback`.
    - Simple API, lower throughput, high latency.
    - Ideal for one-off commands, configuration updates, and heartbeats.
- **Fast-path (Stream-oriented)**: 
    - `producer` / `consumer` rings.
    - High throughput, low latency, asynchronous.
    - Ideal for streaming telemetry, bulk status updates, or high-frequency control.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Foundation**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **Role Setup**:
    - **Server**: `doca_comch_server_create()` $\rightarrow$ configure `max_clients`.
    - **Client**: `doca_comch_client_create()` $\rightarrow$ specify target representor.
3. **Path Selection**: Register either a `recv-callback` (Slow-path) or a `consumer` (Fast-path) before `doca_ctx_start()`.
4. **Connection**: Execute the handshake to move from `DISCONNECTED` $\rightarrow$ `CONNECTED`.
5. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **Messaging**:
    - **Slow-path**: Submit a `doca_comch_task_send` to the PE.
    - **Fast-path**: Write to the `producer` ring / Read from the `consumer` ring.
2. **Drain**: Call `doca_pe_progress()` to trigger the `recv-callback` or update the `consumer` ring.

## 3. Safety & Error Taxonomy

### 3.1. The Representor Visibility Gate
**The most common failure is the DPU not seeing the host representor.**
- If `doca_comch_server_create` returns `DOCA_ERROR_NOT_PERMITTED`:
    - Check if the host is actually connected and the PCIe link is up.
    - Verify the representor is visible via `devlink dev show`.
    - Route to [`doca-setup`](doca-setup.md) to verify hardware bring-up.

### 3.2. Common `DOCA_ERROR_*` Mapping
- **`DOCA_ERROR_AGAIN`**: Returned during `task_send` if the internal buffer is full. Implement a retry loop with `doca_pe_progress()`.
- **`DOCA_ERROR_INVALID_VALUE`**: Incorrect `max_msg_size` or unsupported role for the given device.

## 4. Path Selection: When NOT to use doca-comch
- **High-Volume Data Movement**: For bulk memory transfers $\rightarrow$ use [`doca-rdma`](doca-rdma.md).
- **Packet-level I/O**: For sending raw Ethernet frames $\rightarrow$ use [`doca-eth`](doca-eth.md).
- **Packet Steering**: To route traffic $\rightarrow$ use [`doca-flow`](doca-flow.md).

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md) for PCIe link or role-negotiation failures.
