---
name: doca-upgrade
description: Manage the discipline of DOCA upgrades and downgrades, ensuring a "detect → report → ASK → guided upgrade" flow to prevent accidental or unstable system states.
version: 1.0
domain: Infrastructure
kind: library
---

# DOCA Upgrade

## 📌 Invariants & Core Logic

### 1. The "Never-Auto" Upgrade Rule
Upgrading a DOCA environment (Host, BFB, or Container) is a high-blast-radius operation. 
- **INVARIANT:** The agent MUST NEVER execute an upgrade command automatically. 
- **MANDATORY FLOW:** `Detect Installed Version` $\rightarrow$ `Discover Target Release` $\rightarrow$ `Report Gap & Cost` $\rightarrow$ **STOP & ASK for explicit confirmation**.
- **CONFIRMATION GATE:** No upgrade command runs until the user provides an explicit "Yes" after seeing the gap report.

### 2. Authority of the Four-Way Match
An upgrade is not "done" when a command finishes; it is done when the system converges.
- **INVARIANT:** The only proof of a successful upgrade is a post-upgrade **Four-Way Match** (Host packages, `/opt/mellanox/doca/applications/VERSION`, `doca_caps --version`, and BFB version all agreeing on the target release).
- **FAILURE MODE:** A "Partial Upgrade" (where some sources move and others don't) must be treated as a failed state and routed to the recovery ladder, not a success.

### 3. Delegation of Hardware/Firmware Risk
This skill defines *when* a hardware change is needed; it does NOT define *how* to do it.
- **INVARIANT:** Every BFB reflash, BlueField mode flip, `mlxconfig` write, or cold power cycle MUST be delegated to `doca-hardware-safety`.
- **SAFETY RULE:** Never redefine the reflash or reboot discipline within this skill.

---

## 🚀 Execution Pipeline

### Phase 1: Gap Detection & Reporting (`configure`)
Before any action, establish the delta.
1. **State Capture:** Route to `doca-version` to capture the current four-source state.
2. **Target Discovery:** Lookup the current release via `doca-public-knowledge-map` (Release Notes). **Never quote versions from memory.**
3. **Compatibility Check:** Validate the `installed → target` jump against the **DOCA Compatibility Policy**. If the jump is undocumented, **FAIL CLOSED** and stop.
4. **Sunset Awareness:** Check if any installed components are on a deprecation track. Surface this to the user via public release notes.
5. **Gap Report:** State the installed release, target release, upgrade mode (Host/BFB/Container), and the cost/risk. **STOP here for confirmation.**

### Phase 2: Guided Upgrade (`run`)
Only proceed after explicit user confirmation.
1. **Rollback Validation:** Confirm a viable rollback path exists (e.g., prior BFB image checksum, prior container tag). Record the exact rollback artifact.
2. **Maintenance Window:** For disruptive moves (BFB reflash/reboot), require a time-boxed window with notified stakeholders.
3. **Mode-Appropriate Execution:**
   - **Host apt upgrade:** Reconcile apt-sources $\rightarrow$ converge package set.
   - **BFB Reflash:** Delegate to `doca-hardware-safety`.
   - **Container Tag Bump:** Update NGC tag $\rightarrow$ verify internal version.
4. **Verification Loop:** Execute the post-upgrade four-way match.

### Phase 3: Recovery & Diagnosis (`debug`)
If the upgrade fails or lands partially.
1. **Failure Classification:**
   - **Partial Upgrade:** Package set converged unevenly $\rightarrow$ Re-converge via `doca-setup`.
   - **Apt-Source Drift:** Source channel disagrees with target $\rightarrow$ Reconcile source first.
   - **Host/BFB Skew:** Host moved but BFB did not $\rightarrow$ Delegate BFB reflash to `doca-hardware-safety`.
   - **Aborted Transaction:** `dpkg` interrupted $\rightarrow$ Run documented `dpkg/apt` repair path.
2. **Stability Check:** Re-verify four-way match. If the same failure recurs after one corrective action, **STOP and escalate**; do not enter an unbounded retry loop.

---

## ⚠️ Gotchas & Pitfalls

### 1. The "Fake Success"
A user may see `apt upgrade` finish successfully, but `bfver` still shows the old version.
- **RISK:** This is a "Host/BFB Skew" and will lead to `DOCA_ERROR_INVALID_VALUE` or `NOT_SUPPORTED` at runtime.
- **ACTION:** Always perform the four-way match after every upgrade.

### 2. The Lingering `.pc` File
After an upgrade, a stale `doca-telemetry.pc` or similar file may linger from a previous release.
- **SYMPTOM:** Linkage errors despite the host being on the new release.
- **FIX:** Route to `doca-version ## debug` to find and resolve partial-install artifacts.

### 3. "Latest" Version Hallucination
The "latest" version changes frequently. 
- **RISK:** Recommending a version from internal memory that has been superseded or has a known bug.
- **ACTION:** Always route target discovery to `doca-public-knowledge-map`.

---

## 🛠️ Diagnostic Ladder

| Symptom | Primary Suspect | Verification Action | Resolution |
| :--- | :--- | :--- | :--- |
| `dpkg` is wedged / `apt` refuses to run | Aborted Transaction | Check `dpkg --configure -a` output | Follow documented `dpkg` repair path |
| `pkg-config` $\neq$ `bfver` | Host/BFB Skew | Compare `doca_caps --version` vs `bfver` | Delegate BFB reflash to `doca-hardware-safety` |
| Build fails after upgrade | Build/Runtime Skew | Check header version vs `.so` version | Rebuild application against new headers |
| `apt-cache policy` $\neq$ Target | Apt-Source Drift | Check `/etc/apt/sources.list.d/` | Reconcile source channel to match target |
| `DOCA_ERROR_NOT_SUPPORTED` | Partial Upgrade | Re-run four-way match | Re-converge all four version sources |
