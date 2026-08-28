---
license: Apache-2.0
name: doca-bf4-deployment
description: >
  Expert guide for BlueField-4 (BF4) day-1 platform bring-up via the
  dpu-bmc Redfish surface. Covers Grace OS installation through UEFI HTTP
  Boot, PXE, or Redfish Virtual Media; PLDM firmware updates; cloud-init;
  and a six-layer, evidence-first recovery model.
---

# DOCA BlueField-4 (BF4) Deployment (Ultra-Detailed)

This skill owns the **BMC-driven day-1 lifecycle** for a BlueField-4: a powered but unprovisioned BF4 becomes a Grace OS host with the release-notes target firmware and a verified rollback baseline. BF4 is not BF3: it uses the dpu-bmc, Redfish, PLDM, and a Grace CPU complex—not BF3's RShim/BFB/TMFIFO path. Route BF3 work to `doca-bf3-deployment`.

> **Hardware-change boundary.** A PLDM firmware burn, ISO reflash, power cycle, and BMC factory reset are mutating hardware operations. They can cause an outage, corrupt boot media, or make the DPU unavailable. Before *each* such operation, load `doca-hardware-safety`, show the exact action and its target/blast radius, and obtain explicit confirmation for that action alone. A previous confirmation never authorizes a later reset, burn, or reflash.

## 1. Runtime Contract and Non-Negotiable Rules

### 1.1 Required evidence and inputs
The operator must supply or discover from the live environment:

- An out-of-band reachable BMC, its Redfish endpoint, and a working BMC SSH route to `obmc-console-client`.
- Publicly released bundle ISO, firmware `.fwpkg`, or Grace Ubuntu image as appropriate; an operator-controlled `{iso-uri}`; and release-notes target versions.
- Concrete values for every required action input—credential, URI, BMC inventory value, firmware image, PLDM `{eid}`, or Redfish `{task-id}`—before an upload, attach, burn, reset, or power cycle.
- A maintenance window, pre-change inventory, rollback plan, and OOB recovery path as required by `doca-hardware-safety`.

Never invent credentials, image URIs, device names, Redfish task IDs, EIDs, or firmware version strings. Never print a real password; use placeholders such as `{bmc-user}`, `{iso-uri}`, `{fw-image}`, `{eid}`, and `{task-id}`.

### 1.2 The BF4 version and observability anchors
A green installation is established by agreeing evidence rather than an installer exit code:

| Surface | Evidence | Healthy condition |
| --- | --- | --- |
| OOB console | BMC SSH + `obmc-console-client` | UEFI/installer progresses to a clean Grace login prompt. |
| Firmware task | Returned Redfish Task | `TaskState: Completed`, `PercentComplete: 100`, no exception in `Messages`. |
| Firmware activation | `pldmtool fw_update GetFwParams -m {eid}` | Updated components show the release-notes target as **Active** and no stale Pending value. |
| Inventory cross-check | Redfish `FirmwareInventory` | Per-component running versions agree with PLDM and the captured release-notes target. |
| Grace installation | `cat /etc/mlnx-release` | Installed build matches the target bundle. |
| Virtual media | Redfish `VirtualMedia` resource | `Inserted: false` after installation and before declaring success. |

`pldmtool` is the authoritative Active-versus-Pending activation signal; Redfish `FirmwareInventory` is an independent cross-check. If the two disagree, re-query each **once**. If still inconsistent, stop and escalate with both outputs, the Task resource, the OOB-console capture, and the pre/post-change snapshot. The full DOCA four-way compatibility rule remains owned by `doca-version`.

### 1.3 Safety invariants

1. **OOB before mutation.** No BMC-driven install, firmware update, or recovery begins without a verified OOB console.
2. **Targets come from public release notes.** Do not use remembered/pre-release firmware strings; record the bundle, NIC FW, SBIOS, ERoT, and BMC targets from the matching public release notes.
3. **No blind Task recovery.** A firmware Task that is Running, stalled, or exceptional is high-stakes evidence—not authorization to re-push the bundle or power-cycle.
4. **Virtual media is temporary.** Never eject it mid-install; always detach it after successful verification. Leaving it mounted commonly causes the next boot to re-enter the installer.
5. **No generic factory reset.** A BMC reset is recovery-only: it needs a version-matched public procedure, an operator request, and its own explicit confirmation.

## 2. Execution Pipeline

### 2.1 Configure — establish a safe, observable plan

1. **Prove BMC/OOB access.** Connect to the BMC using operator-provided credentials and reach Grace's serial console with `obmc-console-client`. If that fails, stop and establish OOB access first.
2. **Resolve artifacts and hosting.** Download only public NVIDIA artifacts; host the bundle ISO at `{iso-uri}`. For remote HTTPS Virtual Media, verify unauthenticated access, HTTP `HEAD` support, and a server certificate trusted by the BMC. Do not improvise a certificate-installation command.
3. **Choose exactly one OS-install path.** UEFI HTTP Boot is the default; choose PXE only when existing DHCP/TFTP infrastructure or a custom `bf.cfg` requires it; choose Redfish Virtual Media for fully out-of-band delivery when the installed dpu-bmc version is proven to support it.
4. **Preflight Virtual Media capacity.** For local BMC eMMC hosting, calculate image plus seed payload first and stop when it exceeds **5 GB**. For Grace Ubuntu cloud-init, record the fixed BMC-facing URIs `image.iso` and `config.iso`.
5. **Capture targets and baseline.** Save current `FirmwareInventory`, current `cat /etc/mlnx-release` when Grace already boots, OOB evidence, and release-notes targets. These form the rollback and post-change comparison baseline.
6. **Select optional branches explicitly.** Do not infer a PLDM firmware update or Grace-Ubuntu/cloud-init deployment solely from generation. Select each only when the release notes and operator intent require it.

### 2.2 Build and Modify — routing boundaries

- **Build:** BF4 bring-up has no application or firmware artifact build step. Use published ISO and `.fwpkg` artifacts; assembling either is out of scope. Building a later DOCA application routes to `doca-programming-guide` and then `doca-bare-metal-deployment` or `doca-container-deployment`.
- **Modify:** Firmware configuration changes (including `mlxconfig` writes) are hardware-state changes owned by `doca-hardware-safety`. A new ISO, `.fwpkg`, or cloud-init seed is a fresh bring-up: re-run Configure, the selected Run branch, and Test.

### 2.3 Run — OS installation paths

#### A. UEFI HTTP Boot — recommended
Use when `{iso-uri}` is reachable from the OOB network and no PXE infrastructure is needed.

1. Reach the OOB console and, after action-specific approval, reboot Grace into UEFI.
2. In **Device Manager → Network Device List**, select the correct OOB MAC; set **HTTP Boot Configuration** to `{iso-uri}` and save.
3. In **Boot Manager**, choose the UEFI HTTP entry and watch the installer through the console.
4. On completion, continue with the shared post-install sequence.

#### B. PXE Boot
Use only when DHCP + TFTP infrastructure already exists or a custom `bf.cfg` is required.

1. Verify DHCP, TFTP, and the ISO/menu path before changing Grace boot state.
2. At the OOB console, set the next boot to the OOB IPv4 network device, select the DOCA Arm64 menu and then the ISO.
3. Monitor the complete installer through the OOB console, then perform the shared post-install sequence.

#### C. Redfish Virtual Media
Use for an entirely out-of-band installation after proving dpu-bmc support.

1. For a local BMC upload, reconfirm the ≤5 GB combined payload. Present the exact Redfish `SimpleUpdate` action and obtain explicit approval before transfer.
2. Attach the ISO through Redfish `VirtualMedia` and require `Inserted: true` before booting; Grace should expose it as USB mass storage.
3. Set `BootSourceOverride` to USB (`Once` / UEFI as documented), separately confirm the reset, and monitor via `obmc-console-client`.
4. After install, verify Grace, detach every media object, and require `Inserted: false`.

#### D. Grace Ubuntu plus cloud-init (Virtual Media variant)

1. When customization is needed, create the seed ISO with volume label exactly **`CIDATA`**.
2. Use local eMMC only within the 5 GB combined limit, or a verified remote HTTPS source. Attach under the fixed names **`image.iso`** and **`config.iso`**, regardless of original filenames.
3. Confirm every insertion action, verify `Inserted: true`, then separately confirm USB boot/reset.
4. Verify the operating-system build and intended user, SSH key, and hostname; only then detach the media and verify `Inserted: false`.

#### E. PLDM firmware update
This branch may accompany either OS-install path only when required by the public release notes.

1. Use host-side `flint` only when those notes explicitly require it; its burn and activation are separate mutations.
2. After explicit approval, upload `{fw-image}` through the documented Redfish UpdateService multipart endpoint. Preserve the returned `{task-id}`.
3. Monitor the Task's `TaskState`, `PercentComplete`, and `Messages`. Use its documented retry/timeout guidance if present. Without it, ask the operator to approve a polling interval; take at most **two** observations. If all three values are unchanged across both observations, classify it as stalled and escalate—do not poll indefinitely.
4. With a completed Task, inspect `pldmtool fw_update GetFwParams -m {eid}`. Pending differing from Active is expected before activation.
5. Show and separately obtain confirmation for the required power cycle. Re-run `GetFwParams`; Active must now equal the release-notes target and stale Pending values must be cleared.

### 2.4 Shared post-install and Test — declare success only after the sweep

1. Sign in with the image's documented default credentials and change them immediately; set a strong root password without exposing it in logs.
2. Confirm `cat /etc/mlnx-release` matches the recorded install target and that the OOB console reaches a clean login prompt.
3. For firmware changes, require the completed Redfish Task, PLDM Active targets, and matching Redfish FirmwareInventory. Handle disagreement with the one re-query rule in §1.2.
4. For every Virtual Media path, confirm all media report `Inserted: false` and that Grace no longer enters the installer on a subsequent boot.
5. For cloud-init, verify the expected user, SSH key, and hostname were applied.
6. Save a final snapshot: install method, OS build, per-component firmware versions, Task/PLDM output, and OOB-console evidence. This is the next debug session's baseline.

## 3. Diagnostic Ladder — Six-Layer BF4 Classifier

Evaluate these layers **in order** and preserve their evidence before selecting a recovery. Apply no more than one evidence-backed, non-mutating correction in a sweep; then re-run Test. If the same layer remains non-green, stop and escalate. A separately approved activation power cycle resumes at Test but does not permit another debug loop.

| Layer | State and symptoms | Evidence-first resolution |
| --- | --- | --- |
| **1** | `boot-source-failed`: HTTP URI cannot be fetched, PXE menu absent, or media not in Boot Manager. | Inspect OOB console; confirm URI reachability/served ISO, correct OOB MAC, and that media was attached before reset. |
| **2** | `virtual-media-attach-failed`: `InsertMedia` fails, `Inserted: false`, or Grace has no USB mass-storage device. | Check proven dpu-bmc support, successful transfer, HTTPS HEAD/anonymous access/certificate trust, and local payload ≤5 GB. |
| **3** | `firmware-task-stalled-or-exception`: Redfish Task stays Running, percent stops, or `Messages` contains an exception. | Capture the complete Task and Messages. Do **not** re-push, reset, or power-cycle blindly; escalate using the snapshot and console capture. |
| **4** | `firmware-pending-not-active`: Task completed, but PLDM Pending still differs from Active. | Identify the required power cycle, route it through `doca-hardware-safety`, obtain specific approval, then resume Test. |
| **5** | `cloud-init-not-applied`: OS boots, but intended seed configuration is absent. | Confirm the seed volume is `CIDATA` and both fixed URIs (`image.iso`, `config.iso`) were attached. |
| **6** | `virtual-media-boot-loop`: Grace re-enters the installer after reset. | Detach all Virtual Media, require `Inserted: false`, and clear or re-point the boot override; never eject media during installation. |

## 4. Command and Evidence Reference

| Purpose | Surface / command shape | Required result |
| --- | --- | --- |
| View installer/UEFI | BMC SSH + `obmc-console-client` | Observed boot/installer state; no invented marker. |
| Upload OS/media | Redfish `SimpleUpdate` | Transfer completion before attach. |
| Attach/detach media | Redfish `VirtualMedia` | `Inserted: true` before boot; `false` after verification. |
| Firmware transfer | Redfish UpdateService multipart upload | Returned Task ID retained as evidence. |
| Monitor firmware | Redfish Task: `TaskState`, `PercentComplete`, `Messages` | Completed/100/no exception—or preserved failure evidence. |
| Inspect activation | `pldmtool fw_update GetFwParams -m {eid}` | Active matches target after approved activation. |
| Inventory cross-check | Redfish `FirmwareInventory` | Agrees with PLDM and public release-notes targets. |
| Verify Grace image | `cat /etc/mlnx-release` | Recorded target build string. |

## 5. Deferred Verbs and Handoffs

- **BF3 BFB/RShim/TMFIFO bring-up** → `doca-bf3-deployment`.
- **Running a DOCA-linked binary on healthy Grace** → `doca-bare-metal-deployment`.
- **Deploying DOCA service containers** → `doca-container-deployment`.
- **Grace OS environment preparation (hugepages, pkg-config, devlink, representors)** → `doca-setup`.
- **Hardware-change preflight, maintenance window, rollback, or any mutating recovery** → `doca-hardware-safety`.
- **DOCA version-match analysis** → `doca-version`.
- **Host PCIe/kernel/driver diagnosis** → `doca-debug`.
- **Fleet-scale provisioning or public-doc discovery** → `doca-public-knowledge-map`.
