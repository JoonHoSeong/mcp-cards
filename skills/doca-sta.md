---
name: doca-sta
description: Accelerate the NVMe-over-Fabrics (NVMe-oF) storage target data path on BlueField DPUs using the DOCA STA library.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField, NVMe-oF, Storage Target, RDMA, NVMe-PCI, Storage Acceleration]
---

# Skill Card: doca-sta

## 1. Invariants & Core Model

### Target-Side Acceleration
`doca-sta` is specifically designed to accelerate the **target-side** of an NVMe-over-Fabrics (NVMe-oF) connection.
- **Core Role:** It offloads the NVMe-oF target data path to the BlueField DPU hardware, allowing remote initiators to access local NVMe-PCI disks with minimal host CPU involvement.
- **Invariant:** It is **not an initiator/host transport**. It does not help a client connect to a target; it helps a target serve a client.

### The Target Object Model
The storage hierarchy in `doca-sta` is strictly ordered:
`doca_sta` (Context) $\rightarrow$ `doca_sta_subsystem` (NQN/Namespace) $\rightarrow$ `doca_sta_be` (Backend NVMe-PCI Disk).
- **Invariant:** A namespace must be backed by a `doca_sta_be` backend.
- **Invariant:** The target's identity is defined by the NQN (NVMe Qualified Name) within the `doca_sta_subsystem`.

### Transport Invariant: RDMA-Only
`doca-sta` supports **only the NVMe-over-RDMA transport**.
- **Invariant:** There is no support for NVMe-over-TCP. If the fabric is TCP-based, `doca-sta` cannot be used.

---

## 2. Execution Pipeline

### Phase 1: Capability & Device Audit (`configure`)
1. **Device Support Check:** Call `doca_sta_cap_is_supported` to verify the BlueField DPU supports STA acceleration.
2. **Sizing Limits Query:** Use the `doca_sta_get_max_*` family of functions (e.g., `doca_sta_get_max_qps`, `doca_sta_get_max_io_queue_size`) to determine hardware limits.
3. **RDMA Substrate Prep:** Ensure `doca-rdma` is correctly configured, as STA relies on it for the underlying transport.
4. **Steering Rule Audit:** Verify that `doca-flow` rules are in place to steer incoming NVMe-oF traffic to the STA-managed queues.

### Phase 2: Target Configuration (`configure`)
1. **Context Initialization:** Create a `doca_sta` context and associate it with the target device.
2. **Backend Binding:** Define `doca_sta_be` backend controllers and bind them to local NVMe-PCI disks.
3. **Subsystem Definition:** Create `doca_sta_subsystem` resources, assigning the NQN and defining the namespaces.
4. **Resource Mapping:** Map namespaces to the defined backends.

### Phase 3: Lifecycle Management (`run`)
1. **Context Start:** Invoke `doca_ctx_start()` to activate the STA context.
2. **Connection Acceptance:** Accept incoming NVMe-oF connections from remote initiators.
3. **Queue-Pair Sizing:** Negotiate and size the Admin and I/O queues per connection based on the previously queried hardware limits.
4. **Data Path Activation:** Enable the hardware-accelerated data path for I/O operations.

### Phase 4: Verification & Debugging (`test` / `debug`)
1. **Connectivity Test:** Verify that a remote initiator can successfully perform an "Identify Controller" command.
2. **I/O Validation:** Perform read/write operations and verify data integrity.
3. **Performance Profiling:** Measure I/O queue depth and throughput to ensure acceleration is effective.
4. **Error Analysis:** Map `DOCA_ERROR_*` returns to the STA-specific error taxonomy (e.g., `DOCA_ERROR_IO_FAILED` $\rightarrow$ transport vs. backend failure).

---

## 3. Gotchas & Critical Constraints

### High-Risk Anti-Patterns
- **Confusing Target vs. Initiator:** Attempting to use `doca-sta` on the client/initiator side.
- **Ignoring Queue Limits:** Setting I/O queue depths or counts that exceed the values returned by `doca_sta_get_max_*`, leading to initialization failures.
- **Transport Mismatch:** Attempting to use `doca-sta` for NVMe-over-TCP traffic.
- **Missing Flow Rules:** Failing to program `doca-flow` rules, resulting in NVMe-oF packets being dropped or handled by the slow path, even if STA is `Active`.

### Stability & Hardware Constraints
- **Local Disk Dependency:** `doca-sta` requires physical NVMe-PCI disks attached to the BlueField; it cannot accelerate purely virtual or network-backed disks without an additional backend layer.
- **Memory Alignment:** Improperly aligned buffers for RDMA transfers can lead to `DOCA_ERROR_IO_FAILED`.

---

## 4. Diagnostic Ladder

### Layer 1: Environment & Support
- **Symptom:** `doca_sta_cap_is_supported` returns false.
- **Check:** Is the DOCA version current? Does the hardware support STA?
- **Fix:** Upgrade DOCA or move to a compatible BlueField DPU.

### Layer 2: Initialization & Resource Binding
- **Symptom:** `doca_sta_subsystem` or `doca_sta_be` creation fails.
- **Check:** Are the local NVMe-PCI disks available and not locked by another process? Is the NQN unique?
- **Fix:** Free the local disks or correct the NQN.

### Layer 3: Transport & Connectivity
- **Symptom:** Remote initiator times out during "Identify Controller".
- **Check:** Is the RDMA transport active? Are the `doca-flow` steering rules correct?
- **Fix:** Verify `doca-rdma` status and update `doca-flow` rules.

### Layer 4: I/O & Data Path Failures
- **Symptom:** `DOCA_ERROR_IO_FAILED` during NVMe read/write.
- **Check:** Is there a hardware failure on the backend NVMe disk? Is the RDMA connection unstable?
- **Fix:** Check backend disk health or stabilize the RDMA fabric.

---

## 5. API Reference (C ABI)

| Function / Object | Purpose | Role |
| :--- | :--- | :--- |
| `doca_sta` | The primary context for storage target acceleration. | Context |
| `doca_sta_subsystem` | Defines the target's identity (NQN) and namespaces. | Resource |
| `doca_sta_be` | Backend controller mapping to local NVMe-PCI disks. | Backend |
| `doca_sta_cap_is_supported` | Checks if the device supports STA. | Capability |
| `doca_sta_get_max_*` | Queries hardware limits for queues, subsystems, etc. | Sizing |

## 6. Related Skills
- [`doca-rdma`](../doca-rdma/SKILL.md): The essential RDMA substrate for the NVMe-over-RDMA transport.
- [`doca-flow`](../doca-flow/SKILL.md): The steering surface that directs NVMe-oF traffic to STA queues.
- [`doca-setup`](../../doca-setup/SKILL.md): For initial DOCA installation and device preparation.
- [`doca-version`](../../doca-version/SKILL.md): For version-compatibility checks.
- [`doca-debug`](../../doca-debug/SKILL.md): The cross-cutting debug ladder for all DOCA libraries.
EOF
