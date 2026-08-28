---
name: doca-collectx-deployment
category: devops
description: Use when deploying, configuring, and verifying the CollectX telemetry collector on a DOCA-enabled system.
trigger: "User needs to deploy the CollectX collector, configure telemetry providers and export backends, or troubleshoot a silent telemetry pipeline."
---

# CollectX Deployment (doca-collectx-deployment) Skill Card

## Overview
`doca-collectx-deployment` manages the operational lifecycle of the CollectX telemetry collector. It ensures that telemetry data is correctly sourced from DOCA providers, assembled into schema rows locally, and successfully exported to downstream consumers (e.g., Prometheus, Fluent Bit).

### Core Value Proposition
- **Pipeline Integrity**: Moves from "daemon running" to "consumer receiving rows" via a rigorous end-to-end smoke test.
- **Deterministic Configuration**: Replaces in-place edits with a disciplined "config change = deploy event" workflow.
- **Layered Troubleshooting**: Uses a six-layer error taxonomy to isolate failures (from daemon startup to network transport).
- **Hardware-Awareness**: Integrates with `doca-hardware-safety` for any operation touching device state.

## Implementation Path: The CollectX Lifecycle

### 1. Configuration & Pre-flight (`## configure`)
1. **Provider Selection**: Identify required telemetry providers and their associated counter families.
2. **Hardware Support Probe**: Run a per-device probe to confirm that the selected provider is actually exposed by the target hardware.
3. **Schema Definition**: Define the telemetry schema, including counter identities and sampling cadences.
4. **Backend Configuration**: Configure the export backends (e.g., Prometheus pull, Fluent Bit push, NetFlow) and verify sink reachability.
5. **Baseline Capture**: Record the target DOCA version and the intended deployment contract.

### 2. Launch & Verification (`## run`)
1. **Daemon Launch**: Start the collector daemon using the committed config. Launch mode (foreground, service, container) follows `doca-bare-metal-deployment` or `doca-container-deployment`.
2. **Local Assembly Check**: Confirm the collector is assembling rows locally for enabled providers **before** checking the export.
3. **Backend Emission Check**: Independently verify that every enabled export backend is emitting data.
4. **As-Deployed Snapshot**: Record the DOCA version, device generation, enabled providers, resolved counter identities, and sink endpoints.
5. **Initial Smoke**: Transition to the end-to-end smoke test.

### 3. End-to-End Smoke Test (`## test`)
**This is an iterative loop.** Any mutation (config edit, provider change, backend change) requires a full re-run.
1. **Full Chain Verification**: Confirm: `Device/Source` $\rightarrow$ `Collector Assembly` $\rightarrow$ `Export Backend` $\rightarrow$ `Downstream Consumer`.
2. **Support Re-check**: Re-verify that each committed counter is still exposed by the device.
3. **Schema Alignment**: Confirm schema version agreement between the collector and the consumer.
4. **Load/Cadence Test**: Verify that rows arrive at the configured cadence and the consumer keeps up under load.
5. **Passing Snapshot**: Save the passing configuration as the rollback baseline.

### 4. Layered Triage (`## debug`)
Walk these layers in order; do not skip:
1. **Daemon Startup**: If the collector won't start, check `doca-setup` health and read the daemon's startup output.
2. **Provider Rows**: If the daemon runs but there are no rows for a provider, re-run the support probe. This is the most common "silent drop."
3. **Schema Mismatch**: If rows are malformed, re-derive the schema from the live provider set.
4. **Exporter Silence**: If rows exist locally but nothing ships, verify the backend is enabled and the sink is reachable.
5. **Downstream Skew**: If export ships but the consumer is silent, align schema/counter-identity versions.
6. **Transport/Runtime**: For network or firewall failures, route to `doca-debug`.

## Critical Rules & Safety

### 1. The "Config-as-Deploy" Mandate
**Every edit to the collector configuration is a deployment event.**
- **Rule**: Do not edit config in place. Re-walk the `configure` $\rightarrow$ `run` $\rightarrow$ `test` loop after any change to providers, schema, or backends.

### 2. The "Local-First" Verification Rule
**Always confirm local row assembly before troubleshooting the export backend.**
- **Rule**: If no rows exist locally, the issue is a provider/hardware problem, not an exporter problem.

### 3. The "One-Correction" Limit
**In the `## test` loop, apply at most one corrective mutation before re-testing.**
- **Rule**: If the re-test is still non-green, STOP and escalate to `doca-debug` with the captured evidence.

### 4. The "Mutation-Discipline" Integration
**Any change touching device state must be routed through `doca-hardware-safety`.**
- **Rule**: This skill manages the collector deployment; `doca-hardware-safety` manages the hardware mutation.

## Command Appendix

### CollectX Deployment Invocations
| Purpose | Command/Check | Healthy Output |
| --- | --- | --- |
| Support Probe | Per-device provider probe | Provider exposed and supported on device. |
| Daemon Startup | `collector-daemon --config <path>` | `Started successfully` (in daemon output). |
| Local Assembly | Collector internal logs / metrics | Rows for enabled providers are assembling. |
| Export Health | Downstream consumer (e.g., PromQL) | Expected rows are arriving with correct identities. |
| Schema Version | `collector --version` $\rightarrow$ Consumer | Versions are aligned. |
| Hardware State | `mlxconfig -d <bdf> q` | Matches the expected configuration for the provider. |

## Deferred Topic Boundaries
- **Hardware Counter Reader**: Route to `doca-telemetry` for direct API access.
- **Telemetry Publisher**: Route to `doca-telemetry-exporter` for defining schemas and emitting counters from a program.
- **Productized DTS Container**: Route to the public DTS guide for the packaged NGC container.
- **Environment Prep**: Route to `doca-setup` for installation and hugepages.
- **Hardware Mutation**: Route to `doca-hardware-safety` for firmware/mode changes.
