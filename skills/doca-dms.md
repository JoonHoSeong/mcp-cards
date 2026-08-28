---
name: doca-dms
category: devops
description: Use when deploying and managing the DOCA Device Management Service (DMS) for gNMI/gNOI-based device configuration and telemetry.
trigger: "User wants to configure DMS, set up gNMI/gNOI listeners, manage device persistency, or debug DMS authentication and connectivity issues."
---

# DOCA Device Management Service (doca-dms) Skill Card

## Overview
The DOCA Device Management Service (DMS) provides a standardized interface for managing BlueField DPUs using gNMI (gRPC Network Management Interface) and gNOI (gRPC Network Operations Interface). It acts as a management plane, allowing operators to configure device settings, monitor health, and perform operational tasks via a structured gRPC API.

### Core Value Proposition
- **Standardized Management**: Replaces fragmented CLI tools with a unified, industry-standard gRPC interface (gNMI/gNOI).
- **Programmable Infrastructure**: Enables automation of DPU configuration and telemetry at scale.
- **Secure Access Control**: Supports multiple authentication modes (mTLS, PAM, Localhost-only) to ensure secure management access.
- **Persistence**: Allows configuration settings to survive daemon restarts and system reboots.

## Implementation Path: The DMS Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Deployment Shape Selection**: Choose the deployment topology (e.g., standalone, managed, or integrated) based on the operational environment.
2. **Auth Mode Selection**: Select the appropriate authentication mode:
   - **mTLS**: Mutual TLS for high-security, certificate-based auth.
   - **PAM**: Pluggable Authentication Modules for integrated user management.
   - **Localhost-only**: No authentication; used for local agents or testing (loopback only).
3. **Listener Configuration**: Define the gRPC listener addresses and ports.
4. **User Access Control**: Configure the `-allowed_users` list to restrict access to authorized administrators.

### 2. Execution & Run (`## run`)
1. **Daemon Launch**: Start the `dmsd` (frontend) and `dmspe` (privileged backend) processes.
2. **Verification**: Confirm the daemon is listening on the configured ports (`ss -tlnp`).
3. **Smoke Test**: Execute a simple gNMI `Get` request to verify basic connectivity and response shape.
4. **Persistency Activation**: Enable and verify the configuration-persistency mechanism.

### 3. Verification & Testing (`## test`)
**Testing is an iterative loop; any configuration change triggers a re-sweep.**
1. **Daemon Smoke**: Confirm the daemon answers a documented gNMI `Get`.
2. **Auth Smoke**: Verify that the configured auth mode correctly rejects unauthorized requests.
3. **Persistency Smoke**: Set a value on a test path, restart `dmsd`, and verify the value survives.
4. **Capability Snapshot**: Record the as-deployed gNMI paths and gNOI operations supported by the environment.

## Critical Rules & Safety

### 1. The "Mutation Re-opens Sweep" Rule
**Any change to the DMS configuration (auth mode, listener, user list) must trigger a complete re-run of the `## test` sweep.**
- "It probably still works" is not an acceptable verification state.

### 2. Two-Process Separation
**Maintain the strict separation between `dmsd` (low-priv frontend) and `dmspe` (privileged backend).**
- This architecture is critical for security; never attempt to merge them or bypass the frontend.

### 3. Localhost-only Security Warning
**Localhost-only authentication must NEVER be exposed to an external network.**
- This mode is loopback-only; if it is reachable remotely, the deployment is considered unsafe.

## Diagnostic Ladder (Overlay)

| Layer | DMS Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Transport)** | `UNAVAILABLE` or TLS handshake failures. | Check `systemctl status dmsd`; verify listener address and TLS certificates. |
| **Layer 2 (Auth/Authz)** | `UNAUTHENTICATED` or `PERMISSION_DENIED`. | Verify credentials and `-allowed_users` list; check certificate trust chain. |
| **Layer 3 (Path/Op)** | `INVALID_ARGUMENT` or "operation not supported". | Cross-reference the requested path/operation with the supported-paths list in `CAPABILITIES.md`. |
| **Layer 4 (Backend)** | Nested error from underlying tools (e.g., `mlxconfig`). | Extract the raw tool error; consult the specific tool's documentation. |
| **Layer 5 (Persistency)** | Value does not survive restart. | Confirm recording is enabled; check if the persistency file is writable. |
| **Layer 6 (Library)** | `DOCA_ERROR_*` returned from a wrapped library call. | Route to the matching `libs/<library>` skill for library-specific errors. |

## Command Appendix

### DMS-Specific Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `systemctl status dmsd` | Daemon lifecycle check. | `active (running)` with no restart loops. |
| `journalctl -u dmsd` | Frontend logs. | Requests reaching the daemon; no auth-rejection storm. |
| `ss -tlnp \| grep dmsd` | Port listener verification. | Daemon listening on the documented port. |
| `dmsd --help` | Flag discovery. | Documentation of available configuration flags. |
| `gNMI Get <path>` | Sanity check (documented path). | Returns the expected typed value. |
| `gNMI Set <path>` | Persistency/Mutation test. | Success; subsequent `Get` reflects the change. |

## Deferred Topic Boundaries
- **Management Endpoint Setup**: Preparing the OS/network for DMS is handled by `doca-setup`.
- **Custom App Development**: Building gRPC clients or custom backends is handled by `doca-programming-guide`.
- **Turnkey Telemetry**: Productized telemetry aggregation is handled by the DOCA Telemetry Service (DTS).
