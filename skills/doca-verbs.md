---
name: doca-verbs
category: mlops/inference
description: Use when implementing low-level RDMA/Ethernet operations using the DOCA Verbs library, specifically when higher-level libraries (doca-rdma, doca-eth, doca-rmax) do not expose the required verbs, opcodes, or queue options.
trigger: "User wants to use a specific raw WR flag, raw QP option, SRQ feature, custom CQ-handling pattern, or congestion-control group attachment not available in higher-level DOCA libraries, or is porting libibverbs code to the DOCA Core model."
---

# DOCA Verbs Skill Card

## Overview
DOCA Verbs provides a low-level interface to the underlying network hardware, allowing precise control over Queue Pairs (QP), Completion Queues (CQ), Protection Domains (PD), and Memory Regions (MR). It is the "escape hatch" for advanced users who need control beyond what `doca-rdma`, `doca-eth`, or `doca-rmax` provide.

### The "Drop-Down" Decision
**Do not load this skill unless it is confirmed that a higher-level library cannot satisfy the requirement.**
- **Use `doca-rdma`** for standard Send/Receive/Read/Write/Atomic patterns.
- **Use `doca-eth`** for standard Ethernet queueing.
- **Use `doca-rmax`** for timing-precise media/data-over-IP streaming.
- **Use `doca-verbs` ONLY IF** you need a specific raw Work Request (WR) flag, raw QP attribute, or a custom completion-handling path (e.g., completion channels) not exposed by the above.

### Verbs Object Model (Inside DOCA Core)
Unlike `libibverbs`, `doca-verbs` is integrated into the DOCA Core lifecycle:
- **`doca_verbs_context`**: The root handle for the library.
- **Protection Domain (PD)**: The security boundary for QPs, MRs, and AHs.
- **Completion Queue (CQ)**: Where completions are reported.
- **Queue Pair (QP)**: The primary communication endpoint.
- **Memory Region (MR)**: Registered memory for DMA.
- **Send Request Queue (SRQ)**: Shared receive queue for multiple QPs.
- **Address Handle (AH)**: Used for connectionless communication.
- **Completion Channel**: An optional event-driven delivery mechanism for CQs.

## Implementation Path: The Verbs Lifecycle

### 1. Configuration & Setup
1. **Context**: Create the verbs context via `doca_verbs_context_create`.
2. **Protection Domain**: Create the PD via `doca_verbs_pd_create`.
3. **Completion Channel (Optional)**: Create a completion channel via `doca_verbs_comp_channel_create` if event-driven delivery is required.
4. **Completion Queues (CQ)**: Create CQs via `doca_verbs_cq_create` (using `doca_verbs_cq_attr`). Attach the completion channel here if used.
5. **SRQ / CC Group (Optional)**: Create Shared Receive Queues or Congestion Control groups.
6. **Queue Pair (QP)**:
   - Initialize attributes via `doca_verbs_qp_init_attr_create`.
   - Set PD, CQs, and QP type.
   - Create the QP via `doca_verbs_qp_create`.
7. **QP State Machine**: Transition the QP (e.g., RESET $\rightarrow$ INIT $\rightarrow$ RTR $\rightarrow$ RTS) using `doca_verbs_qp_modify` with `doca_verbs_qp_attr`.
8. **PE Connection**: Connect the context to the Progress Engine (PE) via `doca_pe_connect_ctx` before starting the context.
9. **Start**: Call `doca_ctx_start`.

### 2. Execution & Completion
- **Submission**: Use the `doca_verbs_bridge_post_send` and `doca_verbs_bridge_post_recv` family to submit Work Requests (WRs).
- **Completion Handling (Pick EXACTLY ONE per CQ)**:
    - **DOCA PE (Recommended)**: Drive `doca_pe_progress(pe)`. Completions surface as PE events.
    - **Manual Poll**: Call `doca_verbs_bridge_poll_cq` in a custom loop.
    - **Completion Channel**: `poll`/`epoll` the channel handle $\rightarrow$ `doca_verbs_get_cq_event` $\rightarrow$ `doca_verbs_ack_cq_events` $\rightarrow$ re-arm with `doca_verbs_req_notify_cq`.

### 3. Shutdown
Drain inflight WRs $\rightarrow$ transition QP to ERR $\rightarrow$ `doca_ctx_stop` $\rightarrow$ destroy objects in reverse order (QP $\rightarrow$ SRQ $\rightarrow$ CQ $\rightarrow$ PD $\rightarrow$ Context).

## Critical Rules & Safety

### 1. The "No-Mixing" Rule (HARD BLOCK)
**NEVER mix `libibverbs` handles with `doca-verbs` handles on the same hardware resources.**
- `libibverbs` is the raw kernel uverbs interface; `doca-verbs` is the DOCA Core integrated version.
- Mixing them (e.g., using `ibv_modify_qp` on a `doca-verbs` QP) is unsupported and leads to silent failures or crashes.
- **Resolution**: Port all `libibverbs` code to `doca-verbs` handles.

### 2. Completion Path Integrity
**Pick exactly one completion-handling path per CQ.** Mixing PE-driven, manual poll, and completion channels on a single CQ will result in silently dropped completions.

### 3. Resource Leakage
Always free the device attribute handle returned by `doca_verbs_query_device` using `doca_verbs_device_attr_free` to avoid slow memory leaks.

## Troubleshooting & Recovery

| Error/Symptom | Likely Cause | Resolution |
| --- | --- | --- |
| `DOCA_ERROR_NOT_SUPPORTED` | Requested verb/opcode/attribute not supported by device/firmware. | Run `doca_verbs_query_device` $\rightarrow$ check `doca_verbs_device_attr_get_*`. If false, climb back to a higher-level library. |
| `DOCA_ERROR_INVALID_VALUE` | Bad WR flags or attribute mismatch. | Verify WR construction against headers. Do not assume `libibverbs` field layouts transfer. |
| `DOCA_ERROR_NOT_PERMITTED` | Host-side environment/privilege issue. | Route to `doca-setup ## debug`. Check RDMA stack modules and user groups. |
| `DOCA_ERROR_IO_FAILED` | WR completed with error. | **The submit return is NOT the answer.** Inspect the CQE (Completion Queue Entry) error field via the chosen completion path. |
| `DOCA_ERROR_BAD_STATE` | Lifecycle violation (e.g., modify before create). | Verify order: Configure $\rightarrow$ Start $\rightarrow$ Submit $\rightarrow$ Progress. |

## Command Appendix

### Infrastructure Probing (Precedence: Structured $\rightarrow$ Manual)
Always probe with `doca-env --json` or `doca-capability-snapshot` first. Fall back to manual commands only if structured tools fail.

| Goal | Structured Tool (Prefer) | Manual Fallback | Expected Healthy Output |
| --- | --- | --- | --- |
| Build-time Version | `doca-env --json` | `pkg-config --modversion doca-verbs` | Semver string matching `doca-common`. |
| Linker Flags | N/A | `pkg-config --cflags --libs doca-verbs` | Correct `-I` and `-l` flags (includes `-ldoca_common`). |
| Header Verification | N/A | `ls $(pkg-config --variable=includedir doca-common) \| grep doca_verbs` | List including `doca_verbs.h`. |
| Device Capability | `doca-capability-snapshot` | `doca_caps --list-devs` | Devices with raw verbs capability flags. |
| Runtime Version | `doca-env --json` | `doca_caps --version` | Matches build-time version. |
| Driver Logs | N/A | `sudo dmesg \| tail -n 40` | No `mlx5` or `IB` error messages. |
| Low-level Device State | N/A | `sudo ibv_devinfo` | `state: PORT_ACTIVE` and sane MTU. |
| Trace Logging | N/A | `DOCA_LOG_LEVEL=trace ./binary` | Detailed lifecycle and WR submission logs. |

## Deferred Topic Boundaries
- **General RDMA Work**: Route to `doca-rdma`.
- **General Ethernet Queueing**: Route to `doca-eth`.
- **Media/Data-over-IP**: Route to `doca-rmax`.
- **Installation/Verification**: Route to `doca-setup`.
- **Version Conflicts**: Route to `doca-version`.
- **Hardware Safety/Firmware**: Route to `doca-hardware-safety`.
- **Core Context/PE Internals**: Route to `doca-common`.
