---
name: doca-comm-channel-admin
category: devops
description: Use when using the Comm Channel Admin Tool to inspect, verify, and debug the state of established DOCA Communication Channels (Comch) between a host and a DPU.
trigger: "User needs to inspect the active Comch servers and connections on a host or DPU, or debug a stuck communication channel using the administrative tool."
---

# Comm Channel Admin Tool (doca-comm-channel-admin) Skill Card

## Overview
The `doca-comm-channel-admin` tool is a specialized diagnostic utility used to scan and report the current state of DOCA Communication Channels. Unlike the library API, this tool provides an external, administrative view of the channels, which is critical for isolating whether a communication failure is due to the application logic or the underlying system state.

### Core Value Proposition
- **External State Verification**: Provides an independent view of Comch servers and connections without requiring the application to be running or responsive.
- **System-Level Diagnostics**: Leverages `resourcedump` and MFT to verify that the hardware and driver resources are correctly allocated.
- **Zero-Argument Simplicity**: Operages as a single-purpose scan-and-print tool, eliminating complex command-line arguments.

## Implementation Path: The Admin Tool Workflow

### 1. Execution & State Capture (`## run`)
1. **Binary Verification**: Confirm the binary is located at `/opt/mellanox/doca/tools/doca_comm_channel_admin`.
2. **Zero-Argument Scan**: Run the binary with **zero application arguments**. The tool automatically scans all comch-capable devices.
3. **Artifact Capture**: Capture the complete output:
   - **Exit Status**: Must be 0 for a trustworthy result.
   - **Stdout**: The `SERVERS` and `CONNECTIONS` tables.
   - **Stderr**: Any error messages (e.g., from the internal `resourcedump` call).
4. **Prerequisite Check**: If output is empty or the tool fails, verify that `resourcedump` is in the `PATH` and the user has sufficient privileges.

### 2. End-to-End Verification (`## test`)
1. **Artifact Trust**: Only interpret results from a clean scan (Exit 0, no stderr).
2. **Row Verification**: Locate the expected server or connection in the printed tables.
3. **Cross-Side Agreement**: The administrative view must agree with the program-side connection callback reported by `doca-comch`.
4. **Symmetry Check**: If a row is absent on one side, run the same scan on the opposite side (Host $\leftrightarrow$ DPU) to find the discrepancy.

### 3. Layered Triage (`## debug`)
When a channel is stuck or absent, walk these layers in order:
1. **Tool Installation**: Verify the binary exists. If not, the install profile may be missing the tooling subpackage.
2. **Resource Layer**: Check the `resourcedump` dependency. If this fails, the issue is a setup/privilege problem, not a Comch bug.
3. **Device Binding**: If the scan cannot read specific devices, route the driver/representor diagnostics to `doca-setup`.
4. **Row Discovery**: If the scan is clean but the row is absent, verify that the `CONNECTED` callback has actually fired on the program side.
5. **Version Mismatch**: If the tool's view disagrees with the library API, verify the DOCA version match using `doca-version`.

## Critical Rules & Safety

### 1. The "Zero-Argument" Mandate
**Never add application arguments to the binary.**
- **Rule**: The tool does not support subcommands or device selectors. Adding arguments will result in a failure or incorrect behavior.

### 2. The "Quote-Don't-Paraphrase" Rule
**Exact table output is the primary debug artifact.**
- **Rule**: Quote the `SERVERS` and `CONNECTIONS` tables verbatim. Reformatting them destroys the fidelity needed for downstream system-level debugging.

### 3. The "Prerequisite-First" Rule
**Do not interpret "empty" tables as a lack of connections.**
- **Rule**: Verify `resourcedump` and privileges first. An empty result due to a failed `resourcedump` call is a setup failure, not a state report.

### 4. The "Symmetry" Rule
**A single-side scan is insufficient for full diagnosis.**
- **Rule**: Always be prepared to run the scan on both the Host and the DPU to isolate where the connection state has diverged.

## Command Appendix

### Admin Tool Invocations
| Purpose | Invocation | Owning Step | Healthy Output |
| --- | --- | --- | --- |
| Full State Scan | `/opt/mellanox/doca/tools/doca_comm_channel_admin` | `## run`, `## test`, `## debug` | Exit 0; `SERVERS` and `CONNECTIONS` tables present; no stderr. |

## Deferred Topic Boundaries
- **Installation**: Route to `doca-setup`.
- **Programmatic Creation**: Route to `doca-comch` for the library API.
- **Deep State Inspection**: For internal per-message task completion or queue sizing, route to `doca-comch`.
- **Live Telemetry**: Route to `doca-public-knowledge-map` for the Telemetry Service.
