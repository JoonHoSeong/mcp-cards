---
name: doca-argus
category: devops
description: Use when deploying, configuring, and managing the DOCA Argus security service for anomaly detection and security monitoring on BlueField DPUs.
trigger: "User needs to deploy Argus, configure detection policies, set up the finding-to-SIEM forwarding pipeline, or troubleshoot missing security findings."
---

# DOCA Argus (doca-argus) Skill Card

## Overview
`doca-argus` is a security service that provides anomaly detection and monitoring for BlueField DPUs. It identifies security-relevant events and forwards them to a Security Information and Event Management (SIEM) system for analysis and alerting.

### Core Value Proposition
- **DPU-Native Monitoring**: Monitors the system and workload from the DPU's unique perspective, detecting threats that might be invisible to the host CPU.
- **End-to-End Pipeline**: Manages the entire chain from detection on the DPU $\rightarrow$ forwarding $\rightarrow$ SIEM ingestion.
- **Policy-Driven Detection**: Allows operators to tune detection policies to match their specific workload patterns and reduce false positives.
- **Performance-Aware Security**: Provides sampling knobs to balance security coverage with the host workload's CPU and latency budgets.

## Implementation Path: The Argus Deployment Lifecycle

### 1. Deployment & Lifecycle (`## run`)
1. **Container Deployment**: Deploy the Argus container via the BlueField container manager. Verify the image tag matches the current DOCA release.
2. **Lifecycle Verification**: Confirm the container is `running` and the restart count is stable.
3. **Forwarder Configuration**: Configure the forwarder within the Argus container to reach the SIEM endpoint using the correct authentication material.
4. **Handshake Verification**: Confirm the forwarder has successfully established a handshake with the SIEM destination.

### 2. Detection Policy & Tuning (`## configure`)
1. **Policy Alignment**: Align the detection policy with the workload pattern using the public Argus Service Guide.
2. **Calibration Period**: Establish a baseline of "normal" behavior. Treat early false positives as tuning input, not as bugs.
3. **Sampling Adjustment**: Use the sampling knob to reduce CPU/latency impact on the host workload if performance degradation is observed.
4. **Explicit Disable**: If a detector class must be disabled, do so explicitly with a documented reason and a re-evaluation date. **Do NOT silently disable detectors.**

### 3. End-to-End Validation (`## test`)
**The "Finding-to-SIEM" Loop.**
1. **Local Smoke**: Trigger a known-benign event and confirm a finding is emitted in Argus's local feed.
2. **Forwarding Smoke**: Confirm the same finding identifier appears at the SIEM destination.
3. **Pipeline Baseline**: Establish a steady-state baseline of findings before enabling production alerting on the SIEM side.
4. **Coverage Audit**: Verify that smoke events from all intended host targets are observed and out-of-scope targets are excluded.

### 4. Triage & Debugging (`## debug`)
**The "Argus-Failure" ladder.**
1. **Container Layer**: Check container status, logs, and image tag. If the container is not running, downstream diagnostics are meaningless.
2. **Detection Layer**: If no findings arrive, check the detection policy. If too many arrive, re-tune the policy or sampling knob.
3. **Forwarding Layer**: If findings are local but not in the SIEM, check network reachability and SIEM auth material.
4. **Performance Layer**: If host performance drops, re-tune the sampling knob first, then the detection policy.
5. **Coverage Layer**: If findings are about the wrong targets, re-scope the host-coverage axis in the config.
6. **Version Layer**: Resolve discrepancies between the public guide and the running container via the `doca-version` check.

## Critical Rules & Safety

### 1. The "Container-First" Triage Rule
**Always verify the container's health (running, stable restart count) before diagnosing finding-layer or SIEM-layer issues.**
- **Rule**: A non-running container makes all downstream commands and logs meaningless.

### 2. The "No-Silent-Disable" Mandate
**Never silently disable a detector class to quiet a noise stream.**
- **Rule**: All disables must be explicitly documented with a reason and a re-evaluation date.

### 3. The "Smoke-Before-Bulk" Rule
**One known-benign event must traverse the full pipeline (Argus $\rightarrow$ Forwarder $\rightarrow$ SIEM) before production alerting is enabled.**
- **Rule**: Establish a baseline before trusting the production alert stream.

### 4. The "Production-Default" Path
**Argus (the packaged product) is the production default.**
- **Rule**: Only use the DOCA App Shield library if the user is building a dedicated security product.

## Command Appendix

### Argus Invocations
Use structured helpers first. Fall back to manual commands if probes fail.

| Purpose | Command (class shape) | Healthy Output |
| --- | --- | --- |
| Container Status | BlueField container manager `status` | Container `running`, restart count stable. |
| Container Logs | BlueField container manager `logs` | Startup banner + detector-activation + forwarder-handshake. |
| Local Findings | Argus local feed (API / Dashboard) | Findings emitted at baseline; smoke event visible. |
| Forwarder Status | Log-stream handshake line | Forwarder handshake succeeded. |
| SIEM Ingest | SIEM surface (Splunk / Kibana / etc.) | Smoke event present in the SIEM review surface. |
| Perf Baseline | `top` / `vmstat` / application latency probe | Workload performance within production budget. |
| Image Audit | BlueField container manager `inspect` | Tag matches the release-matched guide. |
| Disable Register | Operator's documented record | Register is current; all disables have reasons/dates. |

## Deferred Topic Boundaries
- **DOCA Installation**: Route to `doca-setup` for environment preparation.
- **SIEM Configuration**: Route to the SIEM's own documentation for ingest-side config (Splunk, etc.).
- **Security Posture Design**: Route to the security ops team for anomaly class and incident response definitions.
- **Custom Security Tooling**: Route to `doca-programming-guide` and the public docs for building with the App Shield library.
- **Other DOCA Services**: Route to `doca-public-knowledge-map` for routing to other services (e.g., `doca-dms`).
