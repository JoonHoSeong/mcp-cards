---
license: Apache-2.0
name: doca-bare-metal-deployment
description: >
  Expert guide for launching, supervising, and debugging DOCA-linked binaries 
  directly on hardware (Host x86 or BlueField Arm). Covers hardware-resource 
  binding, multi-tenant isolation, a 7-layer error taxonomy, and the 
  BlueField platform lifecycle (BFB install, RShim/TMFIFO recovery).
metadata:
  kind: library
  domain: infra
---

# DOCA Bare-Metal Deployment

This skill governs the operational lifecycle of a DOCA-linked application binary running directly on hardware—excluding all containerized (kubelet/static-pod) paths. It is the authoritative procedure for ensuring a binary is correctly bound to hardware resources and isolated from co-tenants.

## I. Invariants & Runtime Contract

### 1. Host & Launch Modes
The deployment contract varies based on the target host mode and the chosen supervision layer:

| Mode | Target | Path |
| :--- | :--- | :--- |
| **Host x86** | Host CPU | DOCA Host install $\rightarrow$ BlueField NIC over PCIe |
| **BlueField Arm** | DPU CPU | DOCA installed on BlueField Arm cores (bare-metal) |

| Launch Mode | Use Case | Observability Surface |
| :--- | :--- | :--- |
| **Direct** | First launch, interactive debug | Shell stdout/stderr |
| **tmux / screen** | Long-running, manual reattach | Detached session pane buffer |
| **systemd** | Production, restart-after-reboot | `journalctl -u <unit> -f` |

### 2. Hardware-Resource Binding
**Rule: Never substitute addresses from memory.** All bindings must be derived from live target output:
- **PCI BDF:** Derived from `lspci -d 15b3:`.
- **NUMA Node:** Root complex owner derived from `numactl --hardware` / `lscpu`.
- **CPU Set:** Pinned to be NUMA-local to the BDF.
- **IRQ Affinity:** Mask mirrored to the CPU set via `/proc/irq/<n>/smp_affinity`.

### 3. Multi-Tenant Isolation
Isolation must be established **BEFORE** the workload starts:
- **Compute/Memory:** cgroup-v2 `cpu.max` / `memory.max` / `io.weight`.
- **Networking:** Per-tenant network namespaces (`ip netns`) containing the specific representor.
- **CPU/NUMA:** Strict `numactl` / `taskset` pinning to prevent cross-NUMA interconnect noise.
- **Hugepages:** Accounting against the cgroup's `memory.max` to prevent `EAL: Cannot get hugepage information`.

### 4. Version Compatibility (The Four-Way Match)
A deployment is only considered "closed" when these four anchors align:
1. **Host/BF DOCA Install:** `pkg-config --modversion doca-common`.
2. **Binary Link-time Version:** Captured during build (`pkg-config doca-*`).
3. **BlueField Firmware:** `flint -d <bdf> q` $\rightarrow$ `FW Version`.
4. **DOCA Version Policy:** The compatibility matrix for the specific release.

### 5. Safety & High-Stakes Policy
- **Smoke Before Bulk:** Trivial-arg $\rightarrow$ Liveness-equivalent $\rightarrow$ Trivial-workload $\rightarrow$ Snapshot $\rightarrow$ Multi-tenant smoke.
- **High-Stakes Failure:** A binary in an auto-restart loop under systemd is **NOT** resilient; it is a delayed diagnosis. Stop the supervisor immediately.
- **Mutating Changes:** Any `mlxconfig set` or BFB reflash MUST route through `doca-hardware-safety`.

---

## II. Execution Pipeline

### 1. Configure (`configure`)
1. **Health Check:** Run `doca-setup ## test`. If failing, route to `doca-setup`.
2. **Host Mode Recognition:** Identify if the binary is `x86_64` (Host) or `aarch64` (BF Arm).
3. **Precondition Close:** Verify Hugepages, `devlink` representors, and `LD_LIBRARY_PATH`.
4. **Launch Mode Selection:** Pick Direct, tmux, or systemd based on the use case.
5. **Binding Capture:** Record live PCI BDF, NUMA node, and CPU set.
6. **Rollback Planning:** Capture pre-launch device state and establish OOB access (BMC/IPMI).
7. **Version Alignment:** Verify the Four-Way Match.

### 2. Run (`run`)
1. **Loader Verification:** `file <binary>` (arch check) and `ldd <binary>` (DOCA library resolution).
2. **Invocation Composition:** Build the command using the captured BDF, NUMA, and CPU pins.
3. **Launch:** Execute in the chosen mode (Direct / tmux / systemd).
4. **Attach Verification:** Check for "no matching device" or "representor not found" in logs.
5. **Liveness Signal:** Verify the per-library trivial-workload signal (e.g., one matched packet for doca-flow).

### 3. Isolation (`isolation`)
*Used when multiple DOCA processes co-tenant on one BlueField.*
1. **Cgroup Budget:** Set `cpu.max` and `memory.max` (including hugepage overhead).
2. **Net-Namespace:** `ip netns add <name>` $\rightarrow$ move representor $\rightarrow$ launch binary in netns.
3. **Binding:** Apply `numactl` / `taskset` strictly local to the BDF's NUMA node.
4. **Verification:** Use `systemd-cgls`, `ip netns exec`, and `taskset -p <pid>` before layering workload.

### 4. Test (`test`)
**The Smoke Loop (Iterative):**
1. **Trivial-Arg:** Run `--help` / `--version`. Verify loader resolution.
2. **Liveness-Equivalent:** Launch with correct bindings but NO workload. Verify clean exit on `SIGTERM`.
3. **Trivial-Workload:** Drive exactly ONE operation. Verify the per-library counter advances by 1.
4. **Snapshot:** Record binary version, DOCA version, FW version, and BDF/NUMA/CPU bindings.
5. **Multi-Tenant Smoke:** Add co-tenants one-by-one; re-verify step 3 after each addition.

### 5. Debug (`debug`)
Walk the **Seven-Layer Taxonomy** in order. Do not skip layers.
1. **Process-won't-start:** Arch mismatch, permissions, or `libdoca_*.so` not found.
2. **Starts-and-exits-immediately:** Missing env vars, config files, or Hugepage failure.
3. **Cannot-find-device:** BDF mismatch, representor not in netns, or eswitch mode error.
4. **Library-error:** `DOCA_ERROR_*` codes. Defer to `doca-debug` and per-library skills.
5. **OOM/Signal/Resource:** `dmesg` OOM-killer, cgroup-v2 budget starvation.
6. **Restart-loop (HIGH-STAKES):** Stop supervisor $\rightarrow$ analyze last log $\rightarrow$ fix root cause.
7. **Co-tenant-noise:** Performance degradation only under multi-tenancy. Check isolation primitives.
- **Final Step:** Cross-cutting host check (Kernel version, driver state, PCIe link health).

---

## III. BlueField Platform Lifecycle

### 1. BFB-Install Lifecycle
1. **Pre-flight:** Capture current BSP version, NIC firmware, RShim daemon state, and OOB path.
2. **`bf.cfg` Authoring:** Define passwords and use `bfb_modify_os()` hooks for SSH keys or `mlxconfig` modes.
3. **Push:** Execute `bfb-install`.
4. **Log Parsing:** **Do not trust exit 0.** Parse RShim console for `[MISC]` / `[ERR]` failures and `DPU is ready` markers.

### 2. RShim & TMFIFO Recovery
- **RShim Check:** Verify `rshim.service` is active and `/dev/rshim*` tree exists.
- **TMFIFO Probe:** Check `ip addr show tmfifo_net0`.
- **The Loopback Trap:** Always run `ip route get <bf-tmfifo-addr>`. If output is `dev lo`, the address is bound locally; the probe is a false positive.
- **Soft-Reset:** Use `rshim` soft-reset only after the state is classified.

### 3. Post-BFB Recovery
1. **Readiness Polling:** Poll for `Linux up` / `DPU is ready` in RShim console; check SSH liveness.
2. **Host PF Rebind:** `modprobe mlx5_core` $\rightarrow$ `echo <bdf> > /sys/bus/pci/drivers/mlx5_core/bind`.
3. **Permission Fix:** Check `stat /home/ubuntu`. If `root:root`, apply `chown -R ubuntu:ubuntu` (requires confirmation).
4. **Log Export:** `scp` Arm-side logs (cloud-init, BSP install) back to the host workspace.
5. **Version Re-close:** Re-verify the Four-Way Match.

### 4. BlueField State Classifier
When the DPU is not healthy, match all applicable states:
1. **`installer-still-running`**: `bfb-install` active + console emitting progress. $\rightarrow$ **WAIT**.
2. **`uefi-only`**: `exit Boot Service` seen, but no `Linux up`. $\rightarrow$ **Cold Power Cycle**.
3. **`linux-up-tmfifo-down`**: `Linux up` seen, but TMFIFO probe fails. $\rightarrow$ **RShim/TMFIFO bring-up**.
4. **`tmfifo-up-ssh-down`**: TMFIFO OK, but SSH hangs/refuses. $\rightarrow$ **Credential re-seed via RShim**.
5. **`arm-ok-host-pfs-unbound`**: SSH OK, but `ip link` / `ibv_devinfo` empty on host. $\rightarrow$ **PF Rebind sequence**.
6. **`host-bf-version-mismatch`**: All healthy, but Four-Way Match fails. $\rightarrow$ **`doca-version` alignment**.

---

## IV. Command Reference

| Purpose | Command Class | Owning Step | Healthy Signal |
| :--- | :--- | :--- | :--- |
| **Binary Arch** | `file <bin>` ; `ldd <bin>` | `run` S1 / `debug` L1 | Arch matches `uname -m`; all `libdoca_*.so` resolved. |
| **PCI Enum** | `lspci -d 15b3:` | `configure` S5 / `debug` L3 | Lists expected BDFs matching launch invocation. |
| **Device Enum** | `devlink dev show` ; `ip link show` | `configure` S3 / `debug` L3 | Expected representors visible and correctly named. |
| **NUMA Topo** | `numactl --hardware` ; `lscpu` | `configure` S5 / `isolation` S3 | Identifies root complex owner node and local CPUs. |
| **CPU Pinning** | `numactl --cpunodebind` / `taskset` | `run` S2 / `isolation` S3 | `numactl --show -p <pid>` matches planned set. |
| **Hugepages** | `mount \| grep huge` ; `cat /proc/meminfo` | `configure` S3 / `debug` L2, L5 | `hugetlbfs` mounted; `HugePages_Free` > 0. |
| **Output (Direct)** | Terminal stdout/stderr | `run` S4 / `debug` L2-5 | Documented bring-up lines; no repeating error lines. |
| **Output (systemd)**| `journalctl -u <unit> -f` | `run` S4 / `debug` L2-6 | Unit `active (running)`; stable restart count. |
| **FW Query (RO)** | `mlxconfig -d <bdf> q` | `configure` S7 | Reports current/next-boot NV-config. |
| **FW Version (RO)** | `flint -d <bdf> q` | `configure` S7 | `FW Version` matches compatibility matrix. |
| **Cgroup Budget** | `systemd-cgls` ; `cat /sys/fs/cgroup/...` | `isolation` S5 / `debug` L5, L7 | Limits match budget; no OOM evidence in `memory.stat`. |
| **Net-Namespace** | `ip netns exec <name> ip link show` | `isolation` S2, S5 / `debug` L3 | Representor is inside the netns the binary launched into. |
| **Host Install Ver**| `pkg-config --modversion doca-common` | `configure` S7 / `debug` L4 | Matches binary link-time version. |
