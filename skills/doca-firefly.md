---
name: doca-firefly
category: devops
description: Deploy and manage the DOCA Firefly service to provide high-precision time synchronization (PTP) on BlueField DPUs, ensuring a disciplined PHC and host clock for time-sensitive workloads.
trigger: "User wants to deploy the Firefly PTP service, configure PTP roles (master/slave), synchronize the BlueField PHC, or troubleshoot time drift in time-sensitive applications."
---

# DOCA Firefly (doca-firefly) Skill Card

## Overview
DOCA Firefly is a containerized service that manages Precision Time Protocol (PTP) on BlueField DPUs. It ensures that the BlueField's PTP Hardware Clock (PHC) is precisely disciplined, which in turn allows the host OS clock to be synchronized with a high-degree of precision. This is critical for workloads like 5G UPF, high-frequency trading, and distributed databases.

### Core Value Proposition
- **High-Precision Sync**: Disciplines the BlueField PHC to a PTP master.
- **Containerized Deployment**: Simplifies the deployment of PTP logic via a managed container.
- **End-to-End Time Chain**: Provides the necessary discipline for the entire chain: PTP Master $\rightarrow$ BlueField PHC $\rightarrow$ Host Clock $\rightarrow$ Consumer Workload.
- **Standardized PTP Profiles**: Supports various PTP profiles and domains to match network requirements.

## Implementation Path: The Firefly Lifecycle

### 1. Deployment & Configuration (`## run`)
1. **Infrastructure Probe**: Confirm BlueField runtime health and verify the available PTP-aware network path.
2. **Container Deployment**:
   - **Image Selection**: Use the container tag that matches the operator's DOCA release as specified in the public Firefly Service Guide.
   - **Config Mapping**: Mount the configuration files to the paths specified in the guide.
3. **Four-Axis PTP Configuration**:
   - **Axis 1: Role/Profile**: Define if the DPU is a master, slave, or boundary clock.
   - **Axis 2: Domain**: Set the PTP domain number to match the upstream master.
   - **Axis 3: Interface**: Configure the correct network interface for PTP traffic.
   - **Axis 4: Transport**: Select the appropriate PTP transport (L2 vs L3).
4. **End-to-End Smoke Test**:
   - **Container Health**: Verify the Firefly container is `running` and stable.
   - **PHC Lock**: Confirm the PHC has acquired a lock and the offset is within the profile's spec using `pmc` or `phc_ctl`.
   - **Host-Follower Sync**: Verify the host OS clock is following the PHC (e.g., via `chronyc tracking` or `ptp4l` status).
   - **Workload Smoke**: Confirm the consumer workload (e.g., Rivermax) can read the disciplined time.

### 2. Triage & Debugging (`## debug`)
**The "Firefly-Debug" Ladder.**
1. **Container Runtime Layer**:
   - **Symptom**: Container restart-loop or image pull failure.
     - **Diagnosis**: Verify image tag and config mount paths against the public guide.
2. **PTP-Config Layer**:
   - **Symptom**: PTP state remains in `LISTENING` or no peers are seen.
     - **Diagnosis**: Walk the "Four-Axis" table (Role, Domain, Interface, Transport). Use `tcpdump` to confirm PTP frames on the wire.
3. **Host-Follower Layer**:
   - **Symptom**: PHC is locked, but the host OS clock drifts.
     - **Diagnosis**: Check the host-side follower (`chrony` / `ptp4l`) configuration. Ensure no competing NTP sources are winning.
4. **PTP-Aware-Path Layer**:
   - **Symptom**: Sync acquired, but offset/jitter is outside spec.
     - **Diagnosis**: Identify non-PTP-aware switches in the network path adding variable latency.
5. **Consumer-Workload Layer**:
   - **Symptom**: PHC and Host Clock are correct, but workload reports drift.
     - **Diagnosis**: Verify the workload is reading the system clock and not its own independent time source.
6. **Version Layer**:
   - **Symptom**: Container behavior disagrees with the Service Guide.
     - **Diagnosis**: Verify the container tag matches the DOCA release version.

## Critical Rules & Safety

### 1. The "End-to-End Chain" Mandate
**PTP is a chain; if any link is broken, the system is undisciplined.**
- **Rule**: Always verify all three legs: (1) PTP Master $\rightarrow$ BlueField PHC, (2) BlueField PHC $\rightarrow$ Host Clock, and (3) Host Clock $\rightarrow$ Consumer Workload.

### 2. The "Guide-as-Truth" Constraint
**Never invent configuration keys, container tags, or PTP flags.**
- **Rule**: All configuration and observability output must come directly from the public Firefly Service Guide.

### 3. The "Path-Selection" Requirement
**Firefly is only for PTP-aware paths.**
- **Rule**: If the network path is not PTP-aware, or if standard NTP/chrony suffices, Firefly is the wrong tool.

### 4. "Smoke before Scale"
**Workload integration must follow a successful end-to-end smoke test.**
- **Rule**: No consumer workload should be deployed until the PHC and host clock are confirmed disciplined.

## Command Appendix

### Firefly & PTP Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Purpose | Command (Class Shape) | Owning Step | Healthy Output |
| --- | --- | --- | --- |
| Container Lifecycle | BlueField container manager start/stop/status | `## run` | Container `running`, restart count stable. |
| Container Logs | BlueField container manager log-stream | `## debug` | PTP state-machine transitions visible. |
| PHC Offset/Freq | `pmc -u -b 0 'GET CURRENT_DATA_SET'` | `## run` | Offset within profile spec. |
| Port State | `pmc -u -b 0 'GET PORT_DATA_SET'` | `## debug` | Slave/master roles reach documented state. |
| Wire Confirmation | `tcpdump` (filter for PTP) | `## debug` | PTP frames egress/ingress as expected. |
| Host-Follower Status | `chronyc tracking` or `pmc` (host-side) | `## run` | Reference ID matches PHC; offset tight. |
| Container Tag | BlueField container manager image-inspect | `## run` | Tag matches DOCA release. |

## Deferred Topic Boundaries
- **Host-Follower Installation**: Installation of `chrony` or `ptp4l` on the host is outside this skill.
- **PTP Topology Design**: Designing the network of boundary clocks and masters is a network-side concern.
- **Consumer Workload Logic**: The internal logic of a PTP-aware app is handled in the specific library skill (e.g., `doca-rmax`).
- **General Installation**: Route to `doca-setup`.
