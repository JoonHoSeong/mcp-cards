---
name: doca-socket-relay
category: devops
description: Use the DOCA Socket Relay tool to bridge host-side socket applications to DPU-side terminators across the host ↔ DPU boundary.
trigger: "User wants to deploy a socket relay, configure forwarding endpoints, verify data plane connectivity between host and DPU, or triage a 'silent break' where connections are accepted but bytes are not delivered."
---

# DOCA Socket Relay (doca-socket-relay) Skill Card

## Overview
`doca-socket-relay` governs the deployment and operational lifecycle of the DOCA Socket Relay tool. It provides a transparent bridge for socket-based traffic between a host application and a target terminator running on a BlueField DPU. This skill ensures that the relay is deployed in the correct shape, configured for reachability, and validated using a rigorous "three-view" inspection.

### Core Value Proposition
- **Transparent Bridging**: Enables legacy or standard socket applications to communicate with DPU-resident services without modifying the application code.
- **Three-Axis Observability**: Enforces a diagnostic discipline that captures the view from the host-app, the relay, and the DPU-terminator to isolate connectivity breaks.
- **Deployment Flexibility**: Supports multiple deployment shapes (Host-side, DPU-side, or Container-based) with tailored configuration and permission rules.
- **Safety-First Validation**: Mandates a "smoke-before-bulk" loop and regression baselining to prevent silent data-plane failures.

## Implementation Path: The Relay Lifecycle

### 1. Configuration (`## configure`)
1. **Deployment Shape Selection**: Determine the appropriate deployment shape based on the target terminator's location and the required trust boundary.
2. **Endpoint Mapping**: Define the host-side listening socket (port/path) and the DPU-side forwarding endpoint.
3. **Privilege Verification**: Ensure the invoking user has the necessary privileges to bind sockets and access network namespaces per the public guide.
4. **Artifact Presence**: Verify the relay binary is present and its version matches the DOCA fabric (four-way match).

### 2. Execution (`## run`)
1. **Bind & Listen**: Invoke the relay to bind to the configured host-side endpoint.
2. **Connectivity Establishment**: Trigger the host application to connect to the relay.
3. **State Inspection**: Monitor the DOCA logger (category `SOCKET_RELAY`) at `INFO` or `DEBUG` level to confirm connection acceptance.
4. **Round-Trip Validation**: Execute a trivial request-response cycle to verify that bytes are flowing through the bridge.

### 3. Validation & Baselining (`## test`)
1. **End-to-End Probe**: Use a known request/response shape to confirm the full path: Host App $\rightarrow$ Relay $\rightarrow$ DPU Terminator $\rightarrow$ Relay $\rightarrow$ Host App.
2. **Topology Verification**: Confirm the forwarding endpoint is reachable and the terminator is active.
3. **Regression Baselining**: Compare the with-relay behavior against a no-relay baseline (where possible) to ensure no unexpected latency or data corruption is introduced.

### 4. Triage & Debugging (`## debug`)
**The "Socket-Bridge" Debug Ladder.**
1. **Artifact Layer**: Verify binary presence and `--help` output. Absence $\rightarrow$ route to `doca-setup`.
2. **Version Layer**: Check for version mismatches between the relay, `doca-common`, and the fabric. Use `doca-version` debug ladder.
3. **Bind Layer**: Triage socket bind failures (port conflicts, permission errors).
4. **Data-Plane Layer (The "Silent Break")**: The relay accepts connections, but bytes never arrive at the DPU.
   - **Action**: Capture the "Three-View" bundle: Relay-side logs, Host-app traces, and DPU-terminator logs.
   - **Diagnosis**: Isolate if the break is at the Relay $\rightarrow$ DPU transition or within the DPU terminator.
5. **Permission Layer**: Triage bind/access errors caused by insufficient user privileges.
6. **Version Drift**: Triage behavior that contradicts the public guide due to version mismatch.
7. **Cross-Cutting Layer**: If the relay is healthy but connectivity fails, escalate to `doca-debug` with the three-view bundle.

## Critical Rules & Safety

### 1. The "Three-View" Mandate
**Never diagnose a relay failure using only one side of the bridge.**
- **Rule**: Every connectivity finding must be backed by a bundle containing the Relay's view, the Host Application's view, and the DPU-side Terminator's view.

### 2. The "No-Paraphrase" Rule
**Quote raw logs; do not summarize the connection state.**
- **Rule**: The agent must quote the `SOCKET_RELAY` logger output verbatim. Paraphrasing loses the fidelity required for precise triage.

### 3. The "Smoke-Before-Bulk" Rule
**State-changing operations re-open the smoke test.**
- **Rule**: Any change to forwarding endpoints or deployment shapes is "high-stakes." It must be followed by a fresh round-trip probe and regression baseline before returning to production use.

### 4. The "Read-Only First" Discipline
**Prefer inspection over restart.**
- **Rule**: Do not recommend "restart the relay" as a first-step fix. Perform a read-only inspection of the three-view bundle first to identify the root cause.

## Command Appendix

### Relay Invocations
Follow the detect $\rightarrow$ prefer $\rightarrow$ fall back $\rightarrow$ report contract.

| Purpose (class) | Invocation (shape) | Owning Step | Healthy Output |
| --- | --- | --- | --- |
| Presence Probe | Binary location + `--help` (or image-inspect for containers) | Configure / Debug L1 | Documented `--help` output is returned. |
| Version Match | `--version` cross-checked with `pkg-config --modversion doca-common` | Configure / Test | Strings agree under the four-way match. |
| Bind Relay | Documented bind invocation (per shape/guide) | Run | Relay reports *bound / listening* signal. |
| Connection Set | DOCA logger output (category `SOCKET_RELAY`) at `INFO`/`DEBUG` | Run / Test / Debug L3 | Log lines show host client connection acceptance. |
| Round-Trip Probe | Trivial host-app request $\rightarrow$ DPU response | Run / Test | All three surfaces report matching byte-in/byte-out events. |
| Baseline Diff | Run same probe without relay (if supported) | Test | Behavior matches between with-relay and no-relay runs. |
| Cross-Cutting Handoff | Bundle (Relay + Host + DPU logs) $\rightarrow$ `doca-debug` | Debug L7 | Downstream ladder consumes bundle as evidence. |

## Deferred Topic Boundaries
- **Fleet Orchestration**: Managing many relays across a fleet is deferred to the operator's platform tooling.
- **Custom Terminator Authoring**: Writing the DPU-side program is deferred to `doca-comch` or `doca-eth`.
- **Container Packaging**: Image builds and pod-spec authoring are deferred to `doca-container-deployment`.
- **DOCA Installation**: Driver and library installation is deferred to `doca-setup`.
- **Control-Plane Programming**: Programmatic host ↔ DPU bridge management is deferred to `doca-comch` and `doca-comm-channel-admin`.
