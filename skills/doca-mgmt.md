# DOCA Management (doca-mgmt)

The `doca-mgmt` library provides a programmatic interface for inspecting and modifying the management-plane state of NVIDIA BlueField-DPUs and ConnectX NICs. It is the primary surface for device-level administration, including capability querying, congestion control global status, diagnostics, and raw firmware-control commands.

## Overview

DOCA Management is a C library (`pkg-config` module: `doca-mgmt`) designed for point-in-time administrative operations. It is **not** intended for high-frequency performance benchmarking or streaming telemetry (use `doca-bench` and `doca-telemetry` for those).

### Core Object Model
- **`doca_mgmt_dev_ctx`**: The top-level device context. Required for all management operations.
- **`doca_mgmt_dev_rep_ctx`**: The representor context. Used for VF/function-level configuration. Created via an existing `dev_ctx`.
- **Sub-domain Handles**: Transient handles used to access specific feature sets:
    - **General Capabilities**: Device/representor attributes (e.g., data-direct).
    - **CC Global Status**: Global enable/disable and protocol selection for Congestion Control.
    - **Diagnostics Data**: Multi-domain diagnostic queries.
    - **ICM Quota**: Management of Internal Connection Map quotas.

### Raw Command Path (`doca_mgmt_raw_cmd`)
The "escape valve" for vendor-documented opcodes. It operates based on a scope ladder to limit the blast radius:
1. `DOCA_MGMT_CMD_SCOPE_CONFIGURATION`: Standard config changes.
2. `DOCA_MGMT_CMD_SCOPE_DEBUG_READ_ONLY`: Safe inspection.
3. `DOCA_MGMT_CMD_SCOPE_DEBUG_WRITE`: Targeted debug modifications.
4. `DOCA_MGMT_CMD_SCOPE_DEBUG_WRITE_FULL`: Wide-scope modifications (Highest risk).

---

## Implementation Path

### 1. Install & Configure
- **Pre-requisite**: DOCA must be installed. This skill assumes the base install exists.
- **Build-time Verification**:
    - Run `pkg-config --modversion doca-mgmt` to confirm library availability.
    - Ensure `doca-common` and `doca-mgmt` share the same semver.
- **Runtime Verification**:
    - Run `doca_caps --list-devs` to identify devices supporting management operations.
    - Verify the host kernel exposes `/dev/fwctl*` character devices (required for raw commands).
    - Confirm the binary has root privileges or the necessary `cap_*` capabilities.

### 2. Build
- **Linker Flags**: Use `pkg-config --cflags --libs doca-mgmt`.
- **Dependencies**: Must link against `-ldoca_mgmt` and `-ldoca_common`.
- **Standard Pattern**: Follow the `doca-programming-guide` Meson build pattern.

### 3. Modify & Use (The Lifecycle)
Every management operation must follow this strict order to avoid leaks and `DOCA_ERROR_INVALID_VALUE`:

1. **Initialize**: `Open Device` $\rightarrow$ `Create doca_mgmt_dev_ctx` $\rightarrow$ `Create doca_mgmt_dev_rep_ctx` (if needed).
2. **Query Capabilities**: Run `_is_supported` on the target sub-domain. Cache this snapshot for the session.
3. **Apply Change (The Modify Workflow)**:
    - **Capture Pre-state**: Call the `_get` / `_query` function. **Hard requirement for rollback.**
    - **Populate Handle**: Use `_set_<field>` accessors to populate all required fields.
    - **Apply**: Call the `_set` / `_modify` function.
    - **Verify**: Immediately re-run the `_get` / `_query` to confirm the device adopted the state.
4. **Teardown**: `Destroy Sub-domain Handles` $\rightarrow$ `Destroy rep_ctx` $\rightarrow$ `Destroy dev_ctx` $\rightarrow$ `Close Device`.

### 4. Test & Verify
- **Experimental API Policy**: All `doca_mgmt_*` symbols are tagged `EXPERIMENTAL`. 
- **Verification Rule**: Because symbols can change between releases, you **must** re-run end-to-end tests against every DOCA upgrade.
- **Authority**: The installed headers (`doca_mgmt.h`, etc.) are the final authority over public documentation.

---

## Debug Ladder

When a call returns `DOCA_ERROR_*` or a write fails to reflect in a re-query, follow this ladder:

### Step 1: Cross-Library Baseline (Refer to `doca-debug`)
Walk the standard ladder: `Install` $\rightarrow$ `Version` $\rightarrow$ `Build` $\rightarrow$ `Link` $\rightarrow$ `Runtime` $\rightarrow$ `Program` $\rightarrow$ `Driver`.

### Step 2: doca-mgmt Overlay (Specifics)
- **Runtime (Layer 5)**:
    - **Order Check**: Did you create `dev_ctx` *before* `rep_ctx`? (Failure $\rightarrow$ `DOCA_ERROR_INVALID_VALUE`).
    - **Privilege Check**: Is the process running as root? (Failure $\rightarrow$ `DOCA_ERROR_OPERATING_SYSTEM`).
    - **Interface Check**: Is `/dev/fwctl*` reachable? (Failure $\rightarrow$ route to `doca-setup`).
- **Program (Layer 6)**:
    - **Field Discipline**: Did you call all required `_set_<field>` accessors before `_set`? (Failure $\rightarrow$ `DOCA_ERROR_BAD_CONFIG`).
    - **Handle Lifecycle**: Are you sharing a transient sub-domain handle across threads without sync?
    - **PCI Format**: Is the address in HEX `Domain:Bus:Device.Function` (e.g., `"0000:3a:00.2"`)? (Failure $\rightarrow$ `DOCA_ERROR_INVALID_VALUE`).

### Step 3: Error Taxonomy & First Actions
| Error | Most Common Cause | First Action |
| --- | --- | --- |
| `INVALID_VALUE` | NULL handle or malformed PCI string | Verify `_create` sequence and PCI HEX format |
| `NOT_SUPPORTED` | Device/Firmware doesn't support feature | Check `_is_supported` result against context |
| `BAD_CONFIG` | Required fields not set via accessors | Ensure all `_set_<field>` calls precede `_set` |
| `IN_USE` | Representor context already initialized | Destroy and re-create representor context |
| `OPERATING_SYSTEM` | `fwctl` ioctl path unreachable | Confirm root privileges and `/dev/fwctl` existence |
| `IO_FAILED` | Firmware rejected the command | Verify four-way version match; check opcode docs |
| `DRIVER` | mlx5 driver/firmware failure | Check `dmesg` and route to `doca-debug` Layer 7 |

---

## Critical Rules

### 1. Hardware Safety Meta-Policy (The Golden Rules)
`doca-mgmt` can modify hardware state. Strict adherence to `doca-hardware-safety` is mandatory:
- **Pre-state Capture is Mandatory**: Never recommend a write (`_set`, `_modify`, `raw_cmd`) without first capturing the current state via `_get`.
- **Narrowest Scope Wins**: Use the most restrictive scope possible. `DEBUG_READ_ONLY` $\rightarrow$ Sub-domain Wrapper $\rightarrow$ `raw_cmd` (last resort).
- **Rollback Requirement**: If a rollback path cannot be documented (via the captured pre-state), the operation **must be refused** and escalated to the operator.

### 2. Operational Discipline
- **Context Isolation**: Every device in a fleet must have its own `doca_mgmt_dev_ctx`. Do **not** share contexts across devices.
- **Raw Command Discipline**: Treat every `raw_cmd` as a firmware-control change. Require: Opcode review $\rightarrow$ Pre-state capture $\rightarrow$ Documented rollback $\rightarrow$ Maintenance window.
- **Teardown Order**: Always destroy child contexts (`rep_ctx`) before parent contexts (`dev_ctx`) to prevent memory leaks.

---

## Command Appendix

| Command | Class of Question | Healthy Output |
| --- | --- | --- |
| `pkg-config --modversion doca-mgmt` | Build-time version check | Semver matching `doca_caps --version` |
| `pkg-config --cflags --libs doca-mgmt` | Linker requirements | Includes path + `-ldoca_mgmt -ldoca_common` |
| `doca_caps --list-devs` | Device compatibility | List of PCIe addresses with capability flags |
| `doca_caps --version` | Runtime DOCA version | Semver matching `pkg-config` output |
| `id -u` / `getcap <bin>` | Privilege verification | `0` (root) or required `cap_*` set |
| `ls /dev/fwctl*` | Kernel interface check | Presence of `/dev/fwctl*` devices |
| `DOCA_LOG_LEVEL=trace ./<bin>` | API call tracing | Trace-level logs for every mgmt-plane call |
| `dmesg \| tail -n 40` | Driver/Firmware errors | No repeated `fwctl` or firmware errors |
| `mlxconfig -d <pcie> q` | Firmware config audit | Config matching the pre-state baseline |
