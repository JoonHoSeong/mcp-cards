---
name: doca-urom
description: Manage the host-side DOCA UROM library to offload remote memory operations (puts, gets, atomics, collectives) from the host CPU onto a BlueField DPU, integrating with HPC / UCX / MPI stacks.
version: 1.0.0
category: software-development
---

# DOCA UROM (Host-Side Library)

The `doca-urom` skill governs the host-side API used to enqueue remote memory operations that are executed by the **DOCA UROM Service** running on a BlueField DPU. It is the critical bridge for HPC/MPI workloads seeking to eliminate host-CPU communication bottlenecks.

##  invariants

- **Paired-Contract Model**: UROM is NOT self-sufficient on the host. Every deployment consists of a **Host Library** (enqueuer) and a **DPU Service** (executor). If the DPU-side service is not running or is at an incompatible version, the library will fail at the first enqueue (typically `DOCA_ERROR_NOT_PERMITTED`), regardless of how "healthy" the host-side `doca_dev` access is.
- **Plugin-Based Capability Surface**: UROM has no static `doca_urom_cap_*` family. The capability surface (what operations are supported) is determined **dynamically at runtime** by calling `doca_urom_service_get_plugins_list` on a started Service. The agent must never quote specific operation symbols from memory; it must route to the discovered plugin list.
- **RDMA Substrate Dependency**: UROM does not replace RDMA; it offloads the *posting* of RDMA work. All UROM operations ride on the underlying RDMA/RoCE/IB fabric. A failing fabric surfaces as `DOCA_ERROR_IO_FAILED` at the UROM API.
- **Two-Tier Context Lifecycle**:
    1. `doca_urom_service`: One per target BlueField. Manages the connection to the DPU service and plugin discovery.
    2. `doca_urom_worker`: One or more per Service. Represents a DPU-side process that executes the specific plugins selected by the host.
- **No Host-CPU Fallback**: If the DPU service is missing or incompatible, the agent must **STOP** and report the service gap. Silently falling back to raw `doca-rdma` creates "silent failures" where offload claims are false.

## Execution Pipeline

### 1. Configure (The Pre-Flight Sequence)
**Goal**: Establish a verified path from Host $\rightarrow$ DPU Service $\rightarrow$ RDMA Fabric.

1. **DPU Service Verification**: Confirm the DOCA UROM Service is deployed and running on the target BlueField (Route to `doca-public-knowledge-map ## DOCA services`).
2. **Version Pairing**: 
    - Host: `pkg-config --modversion doca-urom`.
    - DPU: Obtain version via service guide.
    - Verify match against the [DOCA Compatibility Policy](https://docs.nvidia.com/doca/sdk/doca-compatibility-policy/index.html).
3. **Substrate Health Check**: Verify RDMA port state (`ibv_devinfo` $\rightarrow$ `PORT_ACTIVE`). Route to `doca-rdma ## configure` if fabric is down.
4. **Service Context Setup**:
    - `doca_urom_service_create` $\rightarrow$ `doca_urom_service_set_dev` $\rightarrow$ `doca_ctx_start`.
5. **Plugin Discovery**: Call `doca_urom_service_get_plugins_list`. This is the **authoritative capability snapshot**.
6. **Worker Context Setup**: 
    - `doca_urom_worker_create` $\rightarrow$ `doca_urom_worker_set_service` $\rightarrow$ `doca_urom_worker_set_plugins` (using discovered IDs) $\rightarrow$ `doca_ctx_start`.

### 2. Build & Modify
- **Build**: Use `pkg-config doca-urom`. Transitive dependencies on `libdoca-common` and RDMA libs must be preserved.
- **Modify**: Always start from `/opt/mellanox/doca/samples/doca_urom/`. 
    - **Constraint**: Do not bridge operation families (e.g., don't turn a `put` sample into a `collective` sample); pick the closest matching sample to minimize diff risk.
    - **Constraint**: Retain a "single-pair smoke" (one put/get) even in complex modifications.

### 3. Run & Test (The Iterative Loop)
**Goal**: Prove offload efficacy without skipping layers.

- **Run**: Set `DOCA_LOG_LEVEL=trace`. Ensure the host loop calls `doca_pe_progress()` aggressively; otherwise, enqueues will succeed but completions will never fire.
- **Test Loop**:
    1. **Single-Pair Smoke**: One put + one get $\rightarrow$ verify host completion AND peer-side memory.
    2. **Multi-Op Stress**: 100+ operations $\rightarrow$ check for `DOCA_ERROR_AGAIN` (indicates undersized `max_inflight_tasks`).
    3. **Small Collective**: 2-4 nodes $\rightarrow$ verify collective variant support.
    4. **Full HPC Pattern**: Scale to cluster. Failures here are typically MPI/UCX stack design issues, not UROM bugs.

## Gotchas

- **The `NOT_PERMITTED` Ambiguity**: `DOCA_ERROR_NOT_PERMITTED` at the first enqueue can mean *either* (a) missing `doca_dev` permissions (host-OS fix) *or* (b) DPU-side UROM Service is not running (service-side fix). The agent must check **both** before concluding.
- **The `AGAIN` Signal**: `DOCA_ERROR_AGAIN` is not a hardware error; it is a "queue full" signal. Fix by increasing `doca_urom_worker_set_max_inflight_tasks` or draining the PE more frequently.
- **Teardown Order**: Workers MUST be destroyed before the Service. Out-of-order teardown leads to `DOCA_ERROR_BAD_STATE` and potential DPU-side resource leaks.
- **Symbol Hallucinations**: Exact function names for plugins are install-bound. **NEVER** quote a symbol from memory. Route the user to headers in `$(pkg-config --variable=includedir doca-common)`.

## Diagnostic Ladder

| Signal | Primary Suspect | Verification Action | Escalation Path |
| :--- | :--- | :--- | :--- |
| `DOCA_ERROR_NOT_PERMITTED` (first enqueue) | DPU Service missing/stopped | Check DPU service status via public guide | `doca-public-knowledge-map` $\rightarrow$ Service Guide |
| `DOCA_ERROR_NOT_SUPPORTED` | Version Skew or Plugin Absent | Compare `pkg-config` version vs DPU service version | `doca-version` $\rightarrow$ Compatibility Policy |
| `DOCA_ERROR_IO_FAILED` | RDMA Fabric Failure | Run `ibv_devinfo` and check `dmesg` for `mlx5` errors | `doca-rdma ## debug` |
| `DOCA_ERROR_INVALID_VALUE` | Memory Descriptor Error | Verify local registration and remote handle match | Check `doca_urom_service_set_max_comm_msg_size` |
| `DOCA_SUCCESS` $\rightarrow$ No Completion | PE Progress failure | Check if `doca_pe_progress()` is called in the main loop | `doca-programming-guide` (Core lifecycle) |
| `DOCA_ERROR_AGAIN` | Queue Exhaustion | Check `doca_urom_worker_set_max_inflight_tasks` | Increase queue depth or drain PE |
| `DOCA_ERROR_BAD_STATE` | Lifecycle Violation | Trace call sequence against Core lifecycle | `doca-programming-guide` |
