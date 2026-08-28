# 🎴 Skill Card: doca-hardware-safety (Ultra-Detailed)

**Use this skill as the mandatory safety protocol before performing any "Destructive Operations" on NVIDIA BlueField DPUs or ConnectX NICs.** This includes firmware updates, hardware configuration changes via `mlxconfig`, and mode transitions that could lead to system downtime, loss of management access, or permanent hardware damage ("bricking").

Trigger this skill for requests like *"update BlueField firmware"*, *"change mlxconfig parameters"*, *"flash BFB image"*, *"switch NIC to DPU mode"*, or *"perform a factory reset on the BMC"*.

---

## 1. Core Architecture & Risk Management

Hardware modification in DOCA is not a runtime software change; it involves writing to non-volatile memory (EEPROM/Flash) and altering the hardware's boot-time behavior.

### A. The Destructive Operation Risk Matrix
All hardware changes must be categorized by their risk level to determine the required approval and safety measures.

| Risk Level | Description | Examples | Impact | Mandatory Action |
| :--- | :--- | :--- | :--- | :--- |
| **Level 1: Low** | Runtime configuration changes. | Modifying simple `mlxconfig` runtime parameters, MTU updates. | Temporary network glitch. | Record current state (`mlxconfig -q`). |
| **Level 2: Medium** | Persistence changes requiring reboot. | `mlxconfig` persistent writes, SR-IOV VF count changes, NIC $\leftrightarrow$ DPU mode toggles. | Service outage, boot loops, device invisibility. | Secure Maintenance Window; prepare rollback scripts. |
| **Level 3: High** | Permanent/Irreversible changes. | BFB (BlueField Bundle) re-flashing, direct firmware burning, BMC factory resets, PLDM writes. | Permanent hardware bricking, total loss of RShim/TMFIFO access. | **Administrator Approval**; verify stable BFB version; ensure physical access. |

---

## 2. Implementation Paths: The Safe Execution Workflow

No hardware change should be executed without following these five sequential steps.

### Step 1: Pre-Change Snapshot (Backup)
Establish a verifiable baseline to ensure a return path exists.
- **Config Backup:** `mlxconfig -q > pre_change_config.txt`
- **Capability Snapshot:** Save the output of `doca_caps`.
- **Version Log:** Record current firmware, BFB, and OS versions.

### Step 2: Risk Assessment & Explicit Warning
Identify the risk level and explicitly notify the user of the potential consequences.
- **Example Warning:** *"This operation involves re-flashing the BlueField-3 firmware (Level 3: High Risk). Failure may render the device inaccessible. This must be performed during a maintenance window. Do you wish to proceed?"*

### Step 3: Environmental Isolation
Minimize the blast radius by isolating the target hardware.
- **Service Shutdown:** Terminate all active DOCA applications and background daemons.
- **Traffic Diversion:** Redirect network traffic via LACP/Bonding to ensure the rest of the cluster remains reachable.

### Step 4: Precise Execution
Use deterministic methods to apply the change to prevent typos or partial writes.
- **Scripted Application:** Use `cat` or a shell script rather than manual entry for complex `mlxconfig` strings.
- **Dry Run:** Whenever available, execute the command with a `--dry-run` or validation flag first.

### Step 5: Post-Verification & Restoration
Verify the change and return the system to an operational state.
- **Config Audit:** Run `mlxconfig -q` to confirm the value was written.
- **Reboot & Probe:** Reboot $\rightarrow$ check `lspci` $\rightarrow$ run `doca_caps` to verify device visibility.
- **Traffic Test:** Perform ping tests and validate data plane flow.

---

## 3. Pre-flight Checklist

- [ ] **Baseline Saved:** Is there a `pre_change_config.txt` saved on a remote disk?
- [ ] **Risk Level Identified:** Is this Level 1, 2, or 3?
- [ ] **Maintenance Window:** Is there a scheduled window for potential downtime?
- [ ] **Recovery Image:** (For Level 3) Is the latest stable BFB bundle downloaded and verified?
- [ ] **Physical Access:** (For Level 3) Is there someone on-site to perform a cold boot if the management interface fails?

---

## 4. Critical Rules & Troubleshooting

### A. Operational Golden Rules
1. **No Blind Writes:** Never execute a `mlxconfig` write without first reading the current value.
2. **Sequence Strictness:** Never skip the "Snapshot" phase. A hardware change without a backup is a gamble.
3. **Cold Boot Priority:** If a device becomes unresponsive after a Level 2/3 change, a full physical power cycle (Cold Boot) is the first recovery step before attempting software-based recovery.

### B. Emergency Recovery Matrix

| Symptom | Probable Cause | Resolution Path |
| :--- | :--- | :--- |
| **Device invisible after `mlxconfig`** | Incompatible parameter combination. | Use `pre_change_config.txt` to restore previous values. |
| **Boot failure during BFB flash** | Corrupt image or interrupted write. | Enter RShim/TMFIFO **Recovery Mode** $\rightarrow$ Force re-flash. |
| **Total loss of DPU $\leftrightarrow$ Host comms** | Management channel crash or FW mismatch. | Perform physical Cold Boot $\rightarrow$ Inspect BMC logs via IPMI. |

### C. Debugging Ladder
1. **L1 (Baseline):** Does the current `mlxconfig -q` match the intended target state?
2. **L2 (Visibility):** Does `lspci` see the device? If not, check the PCIe link state.
3. **L3 (Management):** Is RShim responsive? If not, check the BlueField Arm boot logs.
4. **L4 (Hardware):** Are the LED indicators on the DPU showing a fault state (e.g., Red/Amber)?

---

## 5. Final Verification Steps

- [ ] **Config Match:** Does `mlxconfig -q` reflect the exact changes requested?
- [ ] **Functional Match:** Does `doca_caps` report the expected hardware capabilities for the new mode?
- [ ] **Stability Match:** Does the device remain stable under a basic load for 10 minutes post-change?

---

## 6. Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `mlxconfig -q` | Query current HW config | List of parameters and their current values. |
| `mlxconfig -i <dev> set <param>=<val>` | Modify HW config | `Config updated successfully`. |
| `doca_caps` | Verify HW capabilities | List of supported DOCA modules. |
| `lspci -v` | Check PCIe link state | Device listed with correct BDF and link speed. |
