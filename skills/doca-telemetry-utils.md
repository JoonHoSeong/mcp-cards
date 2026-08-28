---
name: doca-telemetry-utils
description: Use the doca_telemetry_utils host-side CLI to discover diagnostic-counter schemas, translate counter names to binary Data IDs, and validate per-device counter support for DOCA Telemetry pipelines.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [telemetry, diagnostics, schema-discovery, data-id, bluefield]
---

# DOCA Telemetry Utils

`doca_telemetry_utils` is the authoritative **operator-side support tool** for DOCA Telemetry. It bridges the gap between human-readable counter names and the binary Data IDs that exporters actually ship.

## 📌 Invariants & Critical Constraints

### 1. Data ID vs. Name (The "Silent Drop" Rule)
- **Invariant**: DOCA Telemetry exporters ship **Data IDs**, not human-readable names.
- **Constraint**: Any counter name placed directly into an exporter configuration without being resolved to a Data ID via this tool is a "recipe for a silent metric drop." The collector will receive nothing, and the exporter will not report an error.

### 2. Schema $\neq$ Device Support
- **Invariant**: The **Schema** (what DOCA knows about) is distinct from **Device Support** (what a specific BlueField at a specific firmware level actually exposes).
- **Constraint**: A counter that resolves cleanly in the schema but is not supported by the target device's hardware/firmware will produce no metrics. **Per-device probing is mandatory before committing any counter to a production config.**

### 3. Read-Only Nature
- **Invariant**: This tool is strictly **read-only**. It does not configure the DOCA Telemetry Service (DTS), modify exporter configs, or ship telemetry. It provides the *knowledge* required to configure those other components honestly.

### 4. Version-Locked Schema
- **Invariant**: Counter names, Data IDs, and property dimensions are tied to the DOCA install version and BlueField firmware.
- **Constraint**: Every counter in an exporter config **must be re-resolved** upon any DOCA version upgrade. A Data ID from version X is not guaranteed to be identical in version Y.

---

## 🚀 Execution Pipeline

### Phase 1: Discovery & Resolution (The Knowledge Path)
1. **Enumerate Schema**: Run `doca_telemetry_utils get-counters` to see every counter the current DOCA install understands.
2. **Identify Property Axes**: Run `doca_telemetry_utils <name>` without properties to see valid options for `node`, `pcie_index`, `depth`, and unit-specific axes.
3. **Resolve to Data ID**: Run `doca_telemetry_utils <name> <prop1=val1> <prop2=val2>` to obtain the binary **Data ID**.

### Phase 2: Hardware Validation (The Safety Gate)
1. **Per-Device Probe**: Run `doca_telemetry_utils <device_PCI> <name> <props>` to verify that the specific target BlueField supports the resolved Data ID.
2. **Commit Decision**: 
   - **Supported**: Add the (Data ID, Name, Props, Device) tuple to the exporter config.
   - **Not Supported**: Reject the counter; select an alternative or check the support matrix in the public guide.

### Phase 3: Deployment & Verification (The End-to-End Loop)
1. **Configure Exporter**: Use the resolved Data IDs in the exporter configuration.
2. **Smoke Test**: Ship a single event $\rightarrow$ verify receipt at the collector.
3. **Reverse-Resolve**: Take the Data ID received by the collector and run `doca_telemetry_utils <DATA_ID>` to confirm it maps back to the intended counter and properties.

---

## ⚠️ Gotchas & Pitfalls

| Pitfall | Symptom | Root Cause | Resolution |
| :--- | :--- | :--- | :--- |
| **Name-based Config** | Exporter "ships" but collector sees nothing. | Config uses `port_rx_bytes` (name) instead of `0x116...` (Data ID). | Resolve name $\rightarrow$ ID via this tool; update config. |
| **Schema-only Resolve** | Counter resolves to an ID, but no data flows. | Counter is known to DOCA but not supported by this specific BlueField/Firmware. | Run the `<device_PCI>` probe; verify support. |
| **Property Hallucination** | `Property-value-out-of-range` error. | Inventing `node` or `depth` values based on generic knowledge. | Run tool without props to see valid options first. |
| **Version Skew** | Data ID resolves to a different name after upgrade. | Counter renamed or property axes changed in new DOCA release. | Re-resolve all counters using `get-counters` on the new install. |
| **Permission Denied** | `doca_dev` binding failure during probe. | Per-device probes require elevated privileges to access PCI hardware. | Run the probe as a user with appropriate privileges (e.g., sudo). |

---

## 🔍 Diagnostic Ladder

**Goal: Resolve "Why is this metric missing/incorrect?"**

1. **Level 1: Tool Availability**
   - Is `doca_telemetry_utils` present in `/opt/mellanox/doca/tools/`?
   - $\rightarrow$ No: Route to `doca-setup` to install Telemetry component.

2. **Level 2: Command Syntax**
   - Is the Data ID `0x`-prefixed hex? Are properties passed correctly?
   - $\rightarrow$ No: Check `--help` or public DOCA Telemetry guide.

3. **Level 3: Schema Validity**
   - Does `get-counters` list the counter name?
   - $\rightarrow$ No: Typo or version mismatch. Check public guide for current version.

4. **Level 4: Property Range**
   - Do the chosen `node`/`pcie_index`/`depth` values exist for this counter?
   - $\rightarrow$ No: Run tool without props $\rightarrow$ pick from the printed list.

5. **Level 5: Hardware Support**
   - Does the `<device_PCI>` probe report "supported"?
   - $\rightarrow$ No: Hardware/Firmware limitation. Do not commit to config.

6. **Level 6: Pipeline Integrity**
   - Tool reports everything is "supported," but collector still sees nothing.
   - $\rightarrow$ Route to `doca-telemetry` (collector side) and `doca-debug` (driver/network).

---

## 🛠️ Command Appendix

**Infra-Aware Preamble**: Always prefer structured helpers (`doca-env --json`, `doca-capability-snapshot`) to verify environment/device flags before calling these manual tools.

| Purpose | Command Shape | Expected Output |
| :--- | :--- | :--- |
| **Enumerate Schema** | `doca_telemetry_utils get-counters` | Full list of all counters DOCA understands. |
| **Check Prop Options**| `doca_telemetry_utils <name>` | List of valid property axes and values for `<name>`. |
| **Resolve Name $\rightarrow$ ID**| `doca_telemetry_utils <name> <props>` | `Data ID: 0x...`, `Name: ...`, `Unit: ...`, `Props: ...` |
| **Hardware Probe** | `doca_telemetry_utils <PCI> <name> <props>`| Resolved tuple + **"supported"** or **"not supported"**. |
| **Reverse Resolve** | `doca_telemetry_utils <DATA_ID>` | Human-readable name, unit, and property values. |

**Audit Tuple Requirement**: Every counter committed to a config must be documented as:
`(DOCA Version, BlueField ID + Firmware, Counter Name, Property Values, Resolved Data ID, Per-Device Probe Result)`
