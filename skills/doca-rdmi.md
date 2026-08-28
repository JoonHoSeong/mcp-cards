---
name: doca-rdmi
description: Program accelerator-initiated one-sided RDMA flows using the DOCA RDMA Initiator (RDMI) library on BlueField DPUs.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField, RDMA, RDMI, DPA, One-Sided, RoCE]
---

# Skill Card: doca-rdmi

## 1. Invariants & Core Model

### RDMA Initiator (RDMI) Definition
DOCA RDMI is a specialized library for **accelerator-initiated one-sided RDMA operations**. It allows a DPA (Data Path Accelerator) kernel to post RDMA writes, reads, or sends directly to a remote responder without host-CPU intervention.
- **Key Distinction:** Unlike `doca-rdma` (which is general-purpose and often host-CPU initiated), `doca-rdmi` is purpose-built for the accelerator-side datapath.
- **Constraint:** It is strictly for **one-sided** operations initiated from the DPA. Two-sided or host-CPU initiated flows must use `doca-rdma`.

### The Object Model
RDMI operations revolve around two primary host-side objects that are later handed off to the DPA:
1. **`doca_rdmi_connection`**: Manages the connection state and provides the interface for attaching completions.
2. **`doca_rdmi_poster`**: Handles the actual posting of work requests to the RDMA hardware.

### DPA Handoff Invariant
The bridge between host-side configuration and DPA-side execution is the **DPA Handle**.
- **Mechanism:** The host calls `doca_rdmi_connection_get_dpa_handle()` or `doca_rdmi_poster_get_dpa_handle()`.
- **Execution:** This handle is passed into the DPA kernel, which then uses the corresponding device-side headers (`doca_rdmi_dev_connection.h`, `doca_rdmi_dev_poster.h`) to drive the hardware.

---

## 2. Execution Pipeline

### Phase 1: Surface Selection (`decide`)
1. **Initiator Audit:** Determine if the RDMA flow is initiated by the DPA kernel (use `doca-rdmi`) or the host CPU (use `doca-rdma`).
2. **Operation Type:** Confirm the flow is one-sided. If two-sided communication is required, route to `doca-rdma`.

### Phase 2: Object Lifecycle (`configure`)
1. **Context Setup:** Initialize the underlying `doca_verbs` context.
2. **Connection Creation:** Create a `doca_rdmi_connection` object.
3. **Completion Wiring:** Attach a `doca_dpa_completion` (for DPA-side polling) or a `doca_verbs_cq` (for host-side polling).
4. **Poster Setup:** Create a `doca_rdmi_poster` for the target flow.
5. **Start Context:** Call `doca_ctx_start()` to activate the RDMI objects on the hardware.

### Phase 3: DPA Integration (`run`)
1. **Handle Retrieval:** Retrieve the DPA-side handles for the connection and poster.
2. **Kernel Injection:** Pass these handles into the DPA kernel during its initialization.
3. **DPA Execution:** The DPA kernel posts work requests using the RDMI device-side API.

### Phase 4: Completion & Audit (`test` & `debug`)
1. **Completion Polling:** Monitor the attached `doca_dpa_completion` for success/failure of the RDMA operations.
2. **Acknowledge (Host):** Use `doca_rdmi_connection_recv_ack()` on the host to acknowledge received completions.
3. **Error Audit:** Check for `DOCA_ERROR_BAD_STATE` or other return codes during the attach/start phase.

---

## 3. Gotchas & Critical Constraints

### High-Risk Anti-Patterns
- **Incorrect Library Choice:** Using `doca-rdma` for a DPA-initiated flow often results in link-time errors or runtime `DOCA_ERROR` due to mismatched handle types.
- **Missing Completion Attach:** Attempting to start the context (`doca_ctx_start`) before attaching a completion queue/context will lead to a failure.
- **DPA-Host Mismatch:** Passing a host-side pointer directly to the DPA kernel instead of the retrieved **DPA Handle**.

### Versioning & Stability
- **EXPERIMENTAL Tag:** Many `doca_rdmi_*` symbols are marked as `EXPERIMENTAL`. Check `pkg-config --modversion doca-rdmi` to verify stability for a given release.
- **Binary Compatibility:** Ensure the `doca-dpacc-compiler` version matches the DOCA SDK version exactly to prevent DPA-side memory corruption.

---

## 4. Diagnostic Ladder

### Layer 1: Setup & Build
- **Symptom:** Link-time "function not found" for `doca_rdmi_*` symbols.
- **Check:** Is `pkg-config doca-rdmi` added to the build flags?
- **Fix:** Update `meson.build` or `Makefile` to include the RDMI library.

### Layer 2: Lifecycle Errors
- **Symptom:** `DOCA_ERROR_BAD_STATE` during `doca_rdmi_connection_dpa_completion_attach`.
- **Check:** Is the connection object already started or destroyed?
- **Fix:** Ensure the attach call happens *before* `doca_ctx_start()`.

### Layer 3: DPA Execution
- **Symptom:** DPA kernel posts work, but the remote responder sees no traffic.
- **Check:** Did the DPA kernel receive the correct DPA handle? Is the remote responder's memory registered (MR) correctly?
- **Fix:** Verify the handle handoff and check the responder's RDMA configuration.

### Layer 4: Completion Failures
- **Symptom:** DPA kernel polls completions indefinitely (hangs).
- **Check:** Is the completion context properly wired to the RDMI connection?
- **Fix:** Re-verify the `doca_rdmi_connection_dpa_completion_attach` sequence.

---

## 5. Command Reference

| Call / Command | Purpose | Location |
| :--- | :--- | :--- |
| `doca_rdmi_connection_create` | Creates the RDMI connection object. | Host API |
| `doca_rdmi_poster_create` | Creates the RDMI poster object. | Host API |
| `doca_rdmi_connection_get_dpa_handle` | Retrieves the handle for DPA kernel use. | Host API |
| `doca_rdmi_connection_recv_ack` | Acknowledges completions on the host side. | Host API |
| `pkg-config doca-rdmi` | Resolves library include/link flags. | Host Shell |

## 6. Related Skills
- [`doca-rdma`](../doca-rdma/SKILL.md): General-purpose RDMA; use for host-CPU initiated or two-sided flows.
- [`doca-dpa`](../doca-dpa/SKILL.md): DPA programming model and `dpacc` compiler.
- [`doca-setup`](../../doca-setup/SKILL.md): Prerequisites and DOCA installation.
- [`doca-debug`](../../doca-debug/SKILL.md): Cross-cutting debug ladder.
- [`doca-gpi`](../doca-gpi/SKILL.md): GPU-side sister skill for RDMA initiation.
EOF
