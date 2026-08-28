---
name: doca-telemetry-exporter
description: Enable structured application telemetry publishing into the DOCA telemetry ecosystem using the DOCA Telemetry Exporter library.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [telemetry, observability, publishing, doca-core]
---

# DOCA Telemetry Exporter: Ultra-Detailed Skill Card

## 1. Invariants & Fundamental Constraints

### 1.1 The Asymmetric Role Split (Publisher vs. Receiver)
The `doca-telemetry-exporter` is strictly a **Publisher (Client)** library. It is linked INTO the user's application to send events. It does NOT:
- Aggregate, persist, or query telemetry.
- Provide a dashboard or a Prometheus scrape endpoint.
- Subscribe to events flowing back into the application.
**The Receiver** is the "DOCA Telemetry Service," a separate DOCA service. Conflating the two is the #1 cause of setup failure.

### 1.2 The Hot-Path Drop-Not-Block Invariant
Telemetry must **never block the application's data path**.
- **Symptom**: `DOCA_ERROR_AGAIN` during an emit call.
- **Meaning**: The transport queue to the receiver is full.
- **Mandatory Behavior**: The application MUST **DROP** the event or push it to a bounded application-side buffer.
- **Forbidden Behavior**: Never implement a blocking retry loop or `sleep()` on `AGAIN`. This destroys the very latency the user is trying to measure.

### 1.3 The Schema-Register-Before-Emit Rule
Every event is shaped by a `doca_telemetry_exporter_schema`.
- **Sequence**: `Exporter Context` $\rightarrow$ `Register Schema` $\rightarrow$ `Emit Event`.
- **Violation**: Emitting against an unknown schema returns `DOCA_ERROR_NOT_FOUND`.

### 1.4 Permission & Staging Model
- **User Permissions**: The exporter runs as the application's normal user. It does **NOT** require `sudo`. 
- **The "Sudo Trap"**: If a user sees `DOCA_ERROR_NOT_PERMITTED`, the cause is almost always the **receiver's transport endpoint permissions**, not the exporter process.
- **Staging Order**: The receiving telemetry consumer MUST be started and reachable **BEFORE** the exporter context is started.

---

## 2. Execution Pipeline

### Phase 1: Role & Path Validation
Before writing code, validate that the exporter is the correct primitive:
1. **Structured Telemetry?** $\rightarrow$ Use Exporter. (If plain logs $\rightarrow$ use `doca_log`).
2. **DOCA Ecosystem Receiver?** $\rightarrow$ Use Exporter. (If custom non-DOCA sink $\rightarrow$ use Prometheus client).
3. **One-way Publishing?** $\rightarrow$ Use Exporter. (If bi-directional communication $\rightarrow$ use `doca-comch`).

### Phase 2: Configuration & Setup
1. **Receiver Readiness**: Verify the receiving service is running.
2. **Capability Discovery**: Call `doca_telemetry_exporter_*_get_*` queries to find the runtime limits for:
   - Max schema fields.
   - Max event size.
   - Supported event types.
   *Runtime queries always override agent memory.*
3. **Schema Definition**: Define `doca_telemetry_exporter_schema` (Field names + Types: Counter, Gauge, or Event).
4. **Schema Registration**: Register all schemas with the exporter context.
5. **Source Creation**: Create `doca_telemetry_exporter_source` objects (one per logical reporter/worker).
6. **Context Activation**: Call `doca_ctx_start()`.

### Phase 3: Build & Linkage
Use the canonical DOCA C/C++ build pattern:
- **Module**: `doca-telemetry-exporter`
- **Cflags**: `pkg-config --cflags doca-telemetry-exporter`
- **Libs**: `pkg-config --libs doca-telemetry-exporter`
- **Version Anchor**: `pkg-config --modversion doca-telemetry-exporter` must match `doca_caps --version`.

### Phase 4: Verification Loop (The Smoke Test)
Do not proceed to bulk emit without this sequence:
1. **Single-Event Smoke**: Emit one event $\rightarrow$ Confirm reception in the receiver's log.
2. **Path Confirmation**: If publish returns success but receiver is empty $\rightarrow$ Check receiver staging/transport.
3. **Bulk Load Test**: Increase rate until `DOCA_ERROR_AGAIN` appears $\rightarrow$ Verify that data-path latency does NOT rise (confirming the drop-not-block behavior).

---

## 3. Gotchas & Pitfalls

| Symptom | Root Cause | Corrective Action |
| :--- | :--- | :--- |
| `DOCA_ERROR_AGAIN` | Transport queue full | **DROP** the event. Do NOT retry/block. |
| `DOCA_ERROR_NOT_FOUND` | Schema/Source not registered | Move registration call before the first emit. |
| `DOCA_ERROR_NOT_PERMITTED` | Receiver endpoint locked | Fix permissions on the receiver's socket/endpoint. Do NOT use `sudo` for the app. |
| `DOCA_ERROR_INVALID_VALUE` | Type mismatch or Payload too large | Compare emit data against the `_get_*` capability snapshot. |
| `DOCA_ERROR_BAD_STATE` | Lifecycle violation | Ensure sequence: `Register` $\rightarrow$ `Start` $\rightarrow$ `Emit`. |
| Success on Emit $\rightarrow$ Empty Dashboard | No reader on the transport | Verify the receiver service is running and listening on the correct transport. |

---

## 4. Diagnostic Ladder

### Layer 1: Role & Environment
- **Question**: "I'm reading about a Telemetry Service, is that this library?"
- **Answer**: No. This is the *Exporter* (Publisher). The *Service* is the Receiver. Route to `doca-public-knowledge-map` for the Service guide.

### Layer 2: Version & Install
- **Check**: `pkg-config --modversion doca-telemetry-exporter` vs `doca_caps --version`.
- **Failure**: Disagreement indicates a partial install. Route to `doca-version ## debug` Layer 2.

### Layer 3: Runtime Permissions
- **Check**: `id` of running user vs ownership of the receiver's transport endpoint.
- **Failure**: `DOCA_ERROR_NOT_PERMITTED`. Fix on the receiver side.

### Layer 4: Program Logic (The "Silent" Failures)
- **Symptom**: `DOCA_ERROR_NOT_FOUND`.
- **Check**: Audit the startup path. Is `doca_telemetry_exporter_schema_register` called before the first emit?

### Layer 5: Transport & Driver
- **Symptom**: `DOCA_ERROR_DRIVER`.
- **Action**: Capture `dmesg | tail -n 40` (sudo). Look for mlx5/socket errors. Route to `doca-setup ## debug` Layer 5.

---

## 5. Command Appendix

| Command | Purpose | Expected Healthy Output |
| :--- | :--- | :--- |
| `pkg-config --modversion doca-telemetry-exporter` | Build-time version check | Semver matching `doca_caps --version`. |
| `pkg-config --cflags --libs doca-telemetry-exporter` | Linker flags | Correct `-I` and `-l` paths for the current install. |
| `id` | Permission verification | User matches the receiver's endpoint allowed list. |
| `DOCA_LOG_LEVEL=trace ./<binary>` | Lifecycle tracing | Trace lines for every `ctx_start` and `emit` call. |
