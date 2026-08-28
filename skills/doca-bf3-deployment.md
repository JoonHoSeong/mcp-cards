---
license: Apache-2.0
name: doca-bf3-deployment
description: >
  Expert guide for BlueField-3 (BF3) day-1 platform bring-up via the classic 
  RShim/BFB path. Manages BFB pushes, TMFIFO recovery, DPU-mode selection, 
  and the six-state BlueField-state-classifier for recovery.
---

# DOCA BlueField-3 (BF3) Deployment (Ultra-Detailed)

This skill manages the "Day-1" lifecycle of a BlueField-3 DPU, taking it from a powered card in a slot to a healthy, verified platform ready for workloads. It focuses on the RShim/BFB transport path; the BMC-Redfish path is handled by `doca-bf4-deployment`.

## 1. Invariants (Runtime Contract)

### 1.1 The RShim/BFB Transport Surface
- **Transport**: All BFB pushes occur via the RShim PCIe/USB interface.
- **Daemon Architecture**: On DOCA 3.3+, RShim is a userspace daemon (`rshim.service`). The absence of a kernel module (`lsmod | grep rshim` is empty) is **EXPECTED** and not a failure.
- **Disambiguation**: In multi-DPU hosts, the agent **MUST** cross-match `/dev/rshim<N>/misc` (`DEV_NAME`) against `lspci -d 15b3: -nn` before any push to avoid flashing the wrong DPU.
- **TMFIFO Defaults**: The management channel defaults are strictly `192.168.100.1/30` (Host) and `192.168.100.2/30` (BF3). The agent never fabricates these subnets.

### 1.2 The Three-Version Anchor (BF3 Overlay)
A BF3 is healthy only if the following three legs align with the four-way match rule:
1. **Host-side**: DOCA-Host version (`pkg-config --modversion doca-common`).
2. **Arm-side**: BFB-image DOCA version (`cat /etc/mlnx-release` + `bfver`).
3. **NIC-side**: Firmware version (`flint -d <bdf> q`). 
*Crucial: `mlxconfig -d <bdf> q` returns configuration, NOT version; it is not a substitute for `flint`.*

### 1.3 Operational Safety
- **The "Exit 0" Trap**: `bfb-install` can exit with code 0 even if the install is partial (e.g., NIC firmware update failed). Success is declared ONLY via the RShim console markers (`Linux up`, `DPU is ready`) and the readiness smoke.
- **OOB Requirement**: A BMC console, serial-over-LAN, or physical UART path is a **mandatory precondition**. Without it, the agent must stop and escalate before any BFB push.
- **Mutating Burns**: BFB reflashes, `mlxconfig set` (mode flips), and firmware burns are destructive. These are governed by `doca-hardware-safety` for preflight/rollback discipline.

## 2. Execution Pipeline

### 2.1 Configure (Pre-flight)
1. **Install-Side Recognition**: Explicitly confirm if the work touches the Host-side (DOCA-Host) or Arm-side (BFB/BlueField-OS).
2. **Host Preconditions**: Verify `rshim` package is installed, `rshim.service` is active, and `/dev/rshim*` nodes are present.
3. **DPU Disambiguation**: Map `/dev/rshim<N>` to the correct physical BDF via `lspci`.
4. **OOB Path Verification**: Confirm availability of a BMC/UART console.
5. **Artifact Validation**: Verify BFB image SHA against the public DOCA Downloads page.
6. **Baseline Capture**: Record current Host DOCA, Arm BFB, and NIC FW versions as the rollback anchor.
7. **Mode Decision**: Decide DPU mode (DPU/Embedded vs. Separated-Host/NIC) to be applied via `bf.cfg` hooks.

### 2.2 Build (Routing Stub)
- No application or BFB artifacts are built here.
- **DOCA Binary $\rightarrow$** `doca-programming-guide`.
- **BFB Image $\rightarrow$** Internal NVIDIA process (Out of Scope).
- **`bf.cfg` $\rightarrow$** Authored at push-time based on the BSP manual schema.

### 2.3 Modify (Platform Change)
Any change to the BFB, `bf.cfg`, or Host-side DOCA is treated as a fresh bring-up event.
1. **Event Trigger**: Identify if the change is a BFB update, `bf.cfg` edit, or Host-side reinstall.
2. **Re-baseline**: Re-walk `configure` step 6 (version anchors).
3. **Re-deploy**: Execute `run` $\rightarrow$ `test` sequence.
4. **Hardware Burns**: Route `mlxconfig set` or firmware burns to `doca-hardware-safety ## modify`.

### 2.4 Run (The BFB Push)
1. **`bf.cfg` Authoring**: Configure password and seed SSH public keys via `bfb_modify_os()` hooks.
2. **BFB Execution**: Run `bfb-install` targeting the disambiguated `/dev/rshim<N>`.
3. **Console Monitoring**: Watch `/dev/rshim<N>/console` for `Linux up` and `DPU is ready`.
4. **TMFIFO Bring-up**: Verify `tmfifo_net0` (or `tm-br`) and run `ip route get 192.168.100.2` to ensure the route is NOT `dev lo` (the local-loopback trap).
5. **Hand-off**: Transition to the readiness smoke.

### 2.5 Test (Readiness Smoke)
Iterative loop: **Verify $\rightarrow$ Recover $\rightarrow$ Verify**.
1. **Readiness Markers**: Poll RShim console and Arm SSH endpoint.
2. **PF Rebind**: If `lspci` shows PFs but `ip link` is empty, route the PF-rebind sequence to `doca-hardware-safety ## modify`.
3. **Install Verification**: Run `cat /etc/mlnx-release` and `bfver` on the Arm side.
4. **Ownership Fix**: Check if `/home/ubuntu` is owned by `root`; if so, apply `chown -R ubuntu:ubuntu /home/ubuntu`.
5. **Version Closure**: Re-close the four-way match per `doca-version`.

## 3. Diagnostic Ladder (The 6-State Classifier)

When a BF3 is not healthy post-push, evaluate these states **IN ORDER**. Report all matches before choosing recovery.

| State | Name | Symptoms | Recovery |
| :--- | :--- | :--- | :--- |
| **1** | `installer-still-running` | `bfb-install` active + console emitting progress. | **WAIT**. Aborting creates the `uefi-only` state. |
| **2** | `uefi-only` | UEFI-exit marker present; no `Linux up` marker. | Capture console/dmesg $\rightarrow$ Cold power cycle $\rightarrow$ Re-push. |
| **3** | `linux-up-tmfifo-down` | `Linux up` present; TMFIFO probe fails. | Check host RShim daemon $\rightarrow$ TMFIFO bring-up $\rightarrow$ Loopback check. |
| **4** | `tmfifo-up-ssh-down` | TMFIFO route OK; SSH refuses/hangs. | Wait for `sshd` bound $\rightarrow$ RShim console credential re-seed. |
| **5** | `arm-ok-host-pfs-unbound` | Arm SSH OK; `lspci` shows PFs but `ip link` is empty. | Route PF-rebind sequence to `doca-hardware-safety ## modify`. |
| **6** | `host-bf-version-mismatch` | All above OK; four-way version match fails. | Resolve apt-source drift $\rightarrow$ Route host reinstall via `doca-setup`. |

**Recovery Rule**: Apply recoveries in dependency order (Installer $\rightarrow$ Boot $\rightarrow$ TMFIFO $\rightarrow$ SSH $\rightarrow$ PF Binding $\rightarrow$ Version).
## 4. Command Reference

### 4.1 RShim & TMFIFO Surface
| Purpose | Command / Path | Healthy Signal |
| :--- | :--- | :--- |
| **Daemon Status** | `systemctl status rshim` | `active (running)` |
| **Device Tree** | `ls /dev/rshim*` | Nodes present |
| **DPU Mapping** | `cat /dev/rshim<N>/misc` -> `DEV_NAME` | Matches `lspci -d 15b3: -nn` |
| **Console Stream** | `cat /dev/rshim<N>/console` | `Linux up` -> `DPU is ready` |
| **TMFIFO Route** | `ip route get 192.168.100.2` | `dev tmfifo_net0` or `dev tm-br` (NOT `dev lo`) |
| **BFB Push** | `bfb-install` (with `.cfg`) | Console markers + Readiness smoke green |

### 4.2 Version & Verification
| Purpose | Command | Target Side | Signal |
| :--- | :--- | :--- | :--- |
| **Host DOCA** | `pkg-config --modversion doca-common` | Host | Version string |
| **Arm DOCA** | `cat /etc/mlnx-release` + `bfver` | Arm | Version string |
| **NIC FW** | `flint -d <bdf> q` | NIC | `FW Version:` line |
| **FW Config** | `mlxconfig -d <bdf> q` | NIC | Config dump (NOT version) |
| **Home Owner** | `stat -c '%U:%G' /home/ubuntu` | Arm | `ubuntu:ubuntu` |

---

## 5. Deferred Verbs & Routing

- **BF4 Bring-up (Redfish path)** -> `doca-bf4-deployment`.
- **Running Binaries (Launch/Binding)** -> `doca-bare-metal-deployment`.
- **Deploying Service Containers** -> `doca-container-deployment`.
- **General Host Install/Env** -> `doca-setup`.
- **Hardware-State Burns** -> `doca-hardware-safety ## modify`.
- **Detailed Versioning Logic** -> `doca-version`.
- **General Debugging (PCIe/Kernel)** -> `doca-debug`.
