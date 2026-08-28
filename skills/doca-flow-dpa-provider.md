---
name: doca-flow-dpa-provider
category: devops
description: Use the `doca-flow-dpa-provider` library to bridge DOCA Flow pipelines to the DPA (Data Path Accelerator), allowing DPA-side kernels to interact with Flow datapath resources.
trigger: "User wants to export a DOCA Flow pipe to the DPA, create DPA-side queues for resource access, or integrate a DPA-side kernel with the DOCA Flow datapath for stateful steering decisions."
---

# DOCA Flow DPA Provider (doca-flow-dpa-provider) Skill Card

## Overview
The `doca-flow-dpa-provider` library acts as the essential bridge between the host-side DOCA Flow API and the DPA (Data Path Accelerator) processor. It allows a host-managed Flow pipeline (pipes and entries) to be "exported" to the DPA, enabling a DPA-side kernel (written in C and compiled with `dpacc`) to read and modify Flow resources (like counters and index-selectors) directly on the DPU, without host intervention.

### Core Value Proposition
- **Low-Latency Datapath Interaction**: Enables the DPA to make steering decisions based on real-time datapath metrics (e.g., per-flow counters) without the latency of a host round-trip.
- **DPA-Side Observability**: Allows DPA kernels to monitor the Flow datapath and apply changes (e.g., redirecting a flow) based on internal DPA logic.
- **Tight Coupling of Lifecycles**: Provides the mechanism to synchronize the host-side Flow pipeline lifecycle with the DPA-side kernel execution.

## Implementation Path: The Provider Integration Lifecycle

### 1. Host-Side Configuration & Export (`## configure`)
1. **Infrastructure Probe**: Use `pkg-config --modversion doca-flow-dpa-provider` and `dpacc --version` to verify that the provider library and the DPA compiler are installed and compatible.
2. **DPA-Capable Device**: Verify the device is DPA-capable via `doca_caps --list-devs`.
3. **Resource Allocation**:
   - Initialize the provider context.
   - **Queue Creation**: Create DPA-side queues (`doca_flow_dpa_queues_create`) BEFORE the first export prepare call. Specify the correct queue types (e.g., `RESOURCES_WRITE` for modifying resources).
4. **Pipe Export Sequence**:
   - **Prepare**: Call `_pipe_export_prepare` to prime the hardware for export.
   - **Export**: Call `_pipe_export` to generate the DPA-side handle.
   - **Address Retrieval**: Call `_pipe_get_device_addr` to obtain the DPA-side address of the pipe.

### 2. Build & Integration (`## build`)
1. **Joint Linker Configuration**: Link the host-side binary with `pkg-config --cflags --libs doca-flow-dpa-provider doca-flow doca-dpa`.
2. **DPA-Side Compilation**: Use `dpacc` to compile the DPA-side kernel. Ensure the kernel's expected launch-argument shape matches the host-side handoff.
3. **Sample Alignment**: Refer to the provided provider samples in `/opt/mellanox/doca/samples/doca_flow/` for the correct integration of host and DPA source.

### 3. Run & Execution (`## run`)
1. **Lifecycle Synchronization**: Ensure the host-side `_pipe_export` is complete before launching the DPA kernel.
2. **DPA-Side Polling**: Implement a strict polling discipline in the DPA kernel to prevent the provider queues from filling up.
3. **Resource Interaction**: Use the DPA-side API to read/modify Flow resources (e.g., `doca_flow_external_resource_memory_read`).
4. **Observability**: Use `DOCA_LOG_LEVEL=trace` to observe the host-side lifecycle transitions.

### 4. Validation & Testing (`## test`)
**The "Resource-Interaction" Validation Pattern.**
1. **Positive Path**: Use a DPA kernel to read a Flow counter, modify it, and verify the change via the host-side `doca-flow` API.
2. **Lifecycle Smoke**: Verify that the host-side provider calls return `DOCA_SUCCESS` in the correct sequence: Queues $\rightarrow$ Prepare $\rightarrow$ Export $\rightarrow$ Address.
3. **DPA-Side Verification**: Use DPA-side tools (from the *DPA Tools* umbrella) to inspect the kernel state and verify it is polling the correct queues.
4. **Sizing Test**: Verify that `_modify_range` and `_memory_read_range` calls are chunked according to the documented limits (32-index for modify, 8-index for read).

### 5. Triage & Debugging (`## debug`)
**The "Provider-Debug" Ladder.**
1. **Infra Layer**: Triage version skew between `doca-flow`, `doca-dpa`, `doca-flow-dpa-provider`, and `dpacc`.
2. **Binding Layer**: Verify BlueField mode and DPA support via `doca_caps`.
3. **Lifecycle Layer (Ordering)**:
   - **Symptom**: `DOCA_ERROR_BAD_STATE`.
     - **Diagnosis**: Check if queues were created before prepare, or if export was called before prepare.
4. **Program Layer (DPA-Side)**:
   - **Symptom**: `DOCA_ERROR_NOT_SUPPORTED` during memory update.
     - **Diagnosis**: Check if the queue config was missing `RESOURCES_WRITE`.
   - **Symptom**: Kernel reads wrong values.
     - **Diagnosis**: Verify the `doca_flow_dpa_addr` matches the current `doca_flow_dpa_ctx`.
   - **Symptom**: `DOCA_ERROR_INVALID_VALUE` during launch.
     - **Diagnosis**: Check for signature mismatch between host and DPA-side launch arguments.
5. **Driver Layer**:
   - **Symptom**: `DOCA_ERROR_DRIVER`.
     - **Diagnosis**: Cross-check against the DOCA Compatibility Policy.
6. **Systemic Layer**: Escalate to `doca-debug` or `doca-setup`.

## Critical Rules & Safety

### 1. The "Lifecycle-Order" Mandate
**Strict ordering of provider calls is required to avoid `DOCA_ERROR_BAD_STATE`.**
- **Rule**: Follow the sequence: Queues $\rightarrow$ Prepare $\rightarrow$ Export $\rightarrow$ Address. Any deviation will result in a state error.

### 2. The "Export-First" Constraint
**Entries must be added to the pipe before the `_pipe_export_prepare` call.**
- **Rule**: If entries are added after the prepare call, the exported pipe will be empty on the DPA side.

### 3. The "Chunking" Contract
**Bulk resource updates must be chunked to avoid saturating the work queue.**
- **Rule**: Use `_modify_range` in 32-index chunks and `_memory_read_range` in 8-index chunks.

### 4. DPA-Side Programming Boundary
**This skill focuses on the provider bridge, not DPA kernel development.**
- **Rule**: For writing DPA-side function bodies or using `dpacc` compiler flags, route the user to the public *DOCA DPA* and *DOCA DPACC Compiler* guides.

## Command Appendix

### Provider Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `pkg-config --modversion doca-flow-dpa-provider` | Verify build-time library version. | Semver string matching `doca_caps --version`. |
| `pkg-config --cflags --libs doca-flow-dpa-provider doca-flow doca-dpa` | Get joint link flags. | Canonical list of include and lib paths. |
| `which dpacc && dpacc --version` | Verify DPA compiler installation. | Version string compatible with the DOCA Compatibility Policy. |
| `doca_caps --list-devs` | Confirm DPA-capable devices. | List of devices exposing DPA support. |
| `DOCA_LOG_LEVEL=trace ./<binary>` | Observe host-side lifecycle transitions. | Trace-level lines for `init`, `queues_create`, `export_prepare`, `export`, `get_device_addr`. |

## Deferred Topic Boundaries
- **DPA Kernel Development**: Route to *DOCA DPA* and *DOCA DPACC Compiler* guides.
- **Flow Pipe Spec Design**: Route to `doca-flow`.
- **General Installation**: Route to `doca-setup`.
- **DPA-Side Debugging**: Route to the public *DPA Tools* umbrella.
