---
name: doca-flow-grpc-server
category: devops
description: Deploy and manage the `doca_flow_grpc` server to provide a gRPC-based remote control plane for DOCA Flow applications, enabling remote management of pipes, counters, and steering decisions.
trigger: "User wants to run the DOCA Flow gRPC server, generate client stubs from .proto files, or manage a DOCA Flow application remotely via gRPC."
---

# DOCA Flow gRPC Server (doca-flow-grpc-server) Skill Card

## Overview
The `doca_flow_grpc` server is a remote control plane that sits on top of the DOCA Flow library. It exposes the internal state and management capabilities of a running DOCA Flow application as a set of gRPC services. This allows external controllers to monitor and modify the datapath (e.g., updating pipes, querying counters) without needing to be linked directly into the application binary.

### Core Value Proposition
- **Remote Management**: Decouples the control plane from the data plane, allowing remote administration of Flow pipelines.
- **Language Agnostic Control**: By using gRPC and `.proto` files, clients can be written in any language supported by gRPC (Python, Go, C++, etc.).
- **Standardized Interface**: Provides a consistent API for interacting with DOCA Flow resources across different deployments.

## Implementation Path: The gRPC Server Lifecycle

### 1. Configuration & Binding (`## configure`)
1. **Infrastructure Probe**: Confirm DOCA install health via `doca-env` and verify the `doca-flow` library version using `pkg-config --modversion doca-flow`.
2. **Server Binary**: Locate the `doca_flow_grpc` binary. Use `--help` to identify the supported flags for address and port binding.
3. **Endpoint Security**:
   - **Bind Strategy**: Bind the server to a trusted, isolated network segment.
   - **External Layer**: Deploy the server behind an external proxy, sidecar, or VPN, as the `doca_flow_grpc` binary itself remains plaintext.
4. **Protocol Contract**:
   - **Proto Location**: Locate the shipped `.proto` files using `pkg-config doca-flow --variable=prefix` followed by a `find` command.
   - **Contract Adherence**: Treat the `.proto` files as the absolute source of truth for RPC method names and message fields.

### 2. Client Generation & Integration (`## build`)
1. **Stub Generation**: Use `protoc` and the appropriate gRPC plugin to generate client stubs from the located `.proto` files.
2. **Client Implementation**: Implement the client logic using the generated stubs, ensuring that request/response messages match the `.proto` contract exactly.
3. **Dialing**: Establish a gRPC channel to the server endpoint through the configured external proxy/VPN.

### 3. Run & Validation (`## run`)
1. **Server Startup**: Launch the `doca_flow_grpc` server with the specified bind address and port.
2. **Smoke Test**:
   - **Dial Test**: Confirm the client can dial the server end-to-end.
   - **Read-Only RPC**: Issue one read-only RPC (e.g., a listing or status call) to confirm the server is successfully wired to the live Flow application.
3. **State Management**:
   - **State-Changing RPCs**: After any RPC that modifies the Flow application state, re-run the read-only smoke test once to confirm the change.
   - **Retry Policy**: Permit only one diagnostic retry after a state-changing action. If it remains non-green, stop and escalate.

### 4. Triage & Debugging (`## debug`)
**The "gRPC-Server-Debug" Ladder.**
1. **Connectivity Layer**:
   - **Server Not Started**: Check server logs and confirm the binary is running.
   - **Bind Failure**: Confirm the bind address/port; refer to the server's own error logs.
2. **External Layer**:
   - **Proxy/VPN Rejection**: Inspect logs from the proxy, sidecar, or VPN. Do not attribute these rejections to the `doca_flow_grpc` server.
3. **RPC Layer**:
   - **Status Codes**: Match the gRPC status code (e.g., `INVALID_ARGUMENT`, `NOT_FOUND`) to the documented contract in the `.proto` files and the `grpc.io` reference.
4. **Application Layer**:
   - **Flow Precondition**: Confirm the underlying DOCA Flow application is ready to accept the RPC.
5. **Version Layer**:
   - **Proto Mismatch**: Verify that the client's `.proto` version matches the server's version.
6. **Systemic Layer**:
   - **Environment**: Hand off to `doca-debug` or `doca-setup` for driver, firmware, or network reachability issues.

## Critical Rules & Safety

### 1. The "Proto-as-Contract" Mandate
**Never invent or paraphrase RPC method names or message fields.**
- **Rule**: Always quote the method and field names exactly as they appear in the shipped `.proto` files.

### 2. The "Plaintext-Endpoint" Warning
**The `doca_flow_grpc` server is plaintext by default.**
- **Rule**: The server MUST be bound to a trusted isolated segment or placed behind a capable external proxy/VPN.

### 3. The "One-Retry" Smoke Policy
**State-changing RPCs re-open the smoke test.**
- **Rule**: After any state-changing RPC, re-run the read-only smoke test once. If the second result is non-green, stop and escalate immediately.

### 4. Status Code Fidelity
**Quote gRPC status codes verbatim.**
- **Rule**: Report the exact gRPC status code (e.g., `FAILED_PRECONDITION`), not a user's prose summary of the error.

## Command Appendix

### Server & Client Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `doca_flow_grpc --help` | Discover the CLI surface and flags. | List of documented flags. |
| `pkg-config --modversion doca-flow` | Confirm Flow library version. | Semver string matching `doca_caps --version`. |
| `pkg-config doca-flow --variable=prefix` $\rightarrow$ `find <prefix> -name '*.proto'` | Locate the shipped `.proto` files. | Absolute paths to `common.proto`, `doca_flow.proto`, etc. |
| `protoc` + gRPC plugin | Generate client stubs from `.proto` files. | Successfully compiled client stubs. |
| `DOCA_LOG_LEVEL=trace ./doca_flow_grpc` | Observe server-side lifecycle and RPC transitions. | Trace-level logs of server activity. |

## Deferred Topic Boundaries
- **DOCA Flow Application Development**: Route to `doca-flow`.
- **General gRPC Tooling**: Route to `grpc.io` for `protoc` installation, language bindings, and auth design.
- **Streaming Telemetry**: Route to the *DOCA Telemetry Service (DTS)* via `doca-public-knowledge-map`.
- **General Installation**: Route to `doca-setup`.
