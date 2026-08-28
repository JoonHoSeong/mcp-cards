---
name: doca-urom-svc
description: Operate the DOCA UROM Service container on BlueField Arm to execute remote memory operations (puts, gets, atomics, collectives) offloaded by a paired host using the `doca-urom` library.
version: 1.0
domain: Infrastructure -> DOCA Services
kind: service
compatibility: BlueField-Arm-only DOCA service container (NGC image); paired with host-side `doca-urom` library.
---

# DOCA UROM Service

## 📌 Invariants & Core Constraints

### 1. The Publisher/Executor Paired-Contract
The DOCA UROM Service is **one half of a strict two-component contract**. It is an **Executor**; it does not generate work.
- **Host Side (Publisher):** Applications link against the `doca-urom` library to enqueue operations.
- **DPU Side (Executor):** The `doca-urom-svc` container receives and executes these enqueued operations against the RDMA fabric.
- **Invariant:** A service running without a paired host library is operationally idle. A host library without a running service results in `doca_ctx_start` or enqueue failures.

### 2. Version Coupling (The Load-Bearing Rule)
The host-side library version and the DPU-side service container tag **must agree** per the DOCA Compatibility Policy.
- **Failure Mode:** Mismatches do not always fail loudly; they often manifest as `DOCA_ERROR_NOT_SUPPORTED` for specific collective families or silent stalls.
- **Verification:** Always cross-check `pkg-config --modversion doca-urom` (Host) and the container image tag (DPU).

### 3. No Standalone Service Authz
The shipped binary has **no internal authorization list** (no `allowed_hosts` or `auth_tokens`).
- **Invariant:** Access is governed exclusively by **DOCA Comch pairing** and the underlying **RDMA permissions/exports**.
- **Security:** A host-visible `DOCA_ERROR_NOT_PERMITTED` is a signal from the Comch/RDMA substrate, NOT a UROM-service internal authz decision.

### 4. Substrate Dependency
The service **consumes** the `doca-rdma` substrate; it does not replace it.
- **Invariant:** RDMA fabric failures (link down, config skew) surface as stalled operations or `DOCA_ERROR_IO_FAILED`. Fixes must be applied to the substrate, not the service config.

---

## 🚀 Execution Pipeline

### Phase 1: Path Selection & Pre-Flight
Before deployment, verify the "Offload Value" and "Hardware Cap":
1. **Value Check:** Ensure the workload pattern (MPI/UCX collectives) actually benefits from DPU offload. If not, maintain the host-CPU path.
2. **Capability Query:** Run `doca_urom_cap_*` queries on the host to confirm the BlueField generation supports the intended UCX components/collectives.
3. **Substrate Gate:**
    - Verify Comch pairing identifies only intended host endpoints.
    - Confirm RDMA exports are narrowly scoped to required memory regions with least-privilege permissions.

### Phase 2: Configuration Axes (Pre-Start)
The operator must commit to these three axes **before** starting the container:
1. **UCX Component/Collective Surface:** Define which primitives the service exposes (bound by BF generation).
2. **Enqueue Queue Depth:** Size the service-side queue based on the intended in-flight depth per host to avoid `DOCA_ERROR_AGAIN`.
3. **Comch Endpoint Pairing:** Configure how the host `doca-urom` library reaches the service via the Comch device/representor.

### Phase 3: Deployment & Activation
1. **Image Acquisition:** Pull the version-matched image from NGC per the DOCA UROM Service Guide.
2. **Environment Setup:**
    - Set `SERVICE_ARGS` for daemon flags (e.g., `--max-msg-size`).
    - Set `UROM_PLUGIN_PATH` for plugin discovery.
    - Mount the `plugins/` directory into the container.
3. **Execution:** Start the container under the BlueField OS container runtime.

### Phase 4: Smoke & Scale
1. **Single-Pair Smoke:**
    - `doca_ctx_start` succeeds on one host.
    - One simple `put`/`get` is observed in service logs.
    - One completion fires on the host's progress engine.
2. **Collective Layering:** Only after the smoke test, enable complex collective patterns (all-reduce, etc.).
3. **Scale:** Point the full HPC workload to the service.

---

## ⚠️ Gotchas & Pitfalls

- **The "Hidden" Version Mismatch:** Upgrading the host library without upgrading the DPU container (or vice versa) is the leading cause of "subtle" failures where some operations work and others return `NOT_SUPPORTED`.
- **Queue Saturation:** If the host sees `DOCA_ERROR_AGAIN` after a burst of successful enqueues, the service-side queue depth is undersized for the workload's in-flight depth.
- **Config Hallucination:** The service is configured via **CLI flags and env vars**, NOT a mounted `.yaml` or `.conf` file. Do not attempt to edit a non-existent config file.
- **One-Service Limit:** Running two UROM service containers on one BlueField is a configuration error; they will compete for the same hardware execution state.

---

## 🔍 Diagnostic Ladder

### Level 1: Container Runtime (The Base)
- **Symptom:** Container fails to start, restart-loops, or `doca_ctx_start` fails immediately.
- **Check:** Image tags, registry credentials, `UROM_PLUGIN_PATH` setting, and `plugins/` mount paths.
- **Fix:** Refer to the public Container Deployment Guide.

### Level 2: Service-Side Resources
- **Symptom:** Host enqueues succeed, but completions never fire, or `DOCA_ERROR_AGAIN` occurs.
- **Check:** Service logs for "queue full" or "handler stuck" messages.
- **Fix:** Increase enqueue queue depth in `SERVICE_ARGS`.

### Level 3: RDMA Substrate
- **Symptom:** Host sees `DOCA_ERROR_IO_FAILED` or substrate counters show increments in error registers.
- **Check:** Use `doca-rdma` tools to verify link state and RoCE/IB config.
- **Fix:** Resolve fabric issues at the substrate level; do not modify service config.

### Level 4: Paired-Version Mismatch
- **Symptom:** `doca_urom_cap_*` says a collective is supported, but runtime enqueue returns `DOCA_ERROR_NOT_SUPPORTED`.
- **Check:** Compare `pkg-config --modversion doca-urom` (Host) vs. container tag (DPU).
- **Fix:** Align versions per the DOCA Compatibility Policy.

---

## 🛠️ Observability Matrix

| Surface | What it Answers | Key Indicator |
| :--- | :--- | :--- |
| **Container Manager** | Is the executor alive? | Status: `Running`, Restart Count: `0` |
| **Service Logs** | Why is the executor failing? | Queue saturation, handler timeouts, version mismatch warnings |
| **Host Progress Engine** | Did the operation actually finish? | Submitted (N) vs. Completed (M) |
| **RDMA Counters** | Is the transport dropping bytes? | Substrate error registers (via `doca-rdma`) |
| **Version Snapshot** | Is the contract valid? | Host ModVersion $\leftrightarrow$ Container Tag match |