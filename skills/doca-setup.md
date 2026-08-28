---
name: doca-setup
category: infra
trigger: Use when a user needs to initialize, configure, or debug the DOCA (Data Center Infrastructure-on-a-Chip Architecture) host environment, or when a new user needs a starting point (no-install path).
description: The "front door" for DOCA deployment. Handles system recognition, environment configuration (hugepages, pkg-config, build tools), and install health verification.
---

# DOCA Setup & Environment Initialization

This skill is the authoritative entry point for any DOCA-related task. It manages the transition from a raw host to a "build-ready" and "run-ready" environment.

## I. Invariants

- **Front-Door Routing**: Every deployment request MUST start with `## recognize` to avoid pushing users onto the wrong path (e.g., pushing a bare-metal user into a container).
- **Deployment Matrix**: Supports four system shapes:
    1. **Host x86 + BlueField NIC**: Standard PCIe connectivity.
    2. **BlueField Arm Bare-Metal**: Running directly on the DPU.
    3. **DPU-only / Converged Accelerator**: Device-only mode (SuperNIC class).
    4. **No-Hardware**: Fresh laptop/VM (routes to NGC Container).
- **Env Core**: The build environment relies on `pkg-config` discovery. Runtime relies on `hugetlbfs` (2MiB pages) and `devlink` representors.
- **Version Coherence**: The `pkg-config --modversion doca-common` must align with the `doca_caps --version` and the BFB firmware version.
- **Safety**: Hardware state changes (e.g., eswitch mode) require explicit consent as they disrupt active flows.

## II. Execution Pipeline

### 1. `recognize` (The Front Door)
**Goal**: Detect system shape and route to the correct deployment skill.

- **Step 0: Scale Gate**: If the user mentions "fleet-scale", "production racks", or "K8s operator", route immediately to `doca-public-knowledge-map` (DPF/Network Operator) and stop.
- **Step 1: Auto-Detect**: Run the following and quote results to the user:
    - `uname -m` (x86_64 vs aarch64).
    - `lspci -d 15b3:` (BlueField visibility).
    - `pkg-config --modversion doca-common` (Install presence).
    - `cat /etc/nvidia/bf-release` (BF Arm marker).
- **Step 2: Residual Questions**: Ask only what is missing:
    - **Target**: Host CPU, BF Arm, DPU-only, or unsure?
    - **Packaging**: Kubelet-standalone container or Bare-metal binary?
    - **Workload**: Custom App or NVIDIA Service (Argus/DMS/etc.)?
- **Step 3: Routing**:
    - **No-Hardware/No-Install** $\rightarrow$ `## no-install`.
    - **Host x86 / BF Arm + Service + Container** $\rightarrow$ `doca-container-deployment`.
    - **Host x86 / BF Arm + App + Bare-Metal** $\rightarrow$ `doca-bare-metal-deployment`.
    - **DPU-only** $\rightarrow$ `doca-bare-metal-deployment` (+ `doca-hardware-safety`).
- **Step 4: Confirm**: Restate: *"You are on a `<shape>`, deploying a `<kind>` via `<path>`; I'll continue with `<skill>`."*

### 2. `configure` (Environment Preparation)
**Goal**: Transform a DOCA-installed host into a build/run-ready state.

- **Pre-check**: Verify Apt repo consistency and OS point release match via `doca-version`.
- **Phase 1: Build-Tool Baseline**:
    - **Ubuntu/Debian**: Ensure `build-essential`, `meson`, `ninja-build`, `cmake`, `pkg-config`, `libjson-c-dev`, `liblz4-dev` are installed.
    - **RHEL/OEL**: Ensure `@Development Tools`, `meson`, `ninja-build`, `cmake`, `pkgconfig`, `json-c-devel`, `lz4-devel` are installed.
- **Phase 2: Discovery Wiring**:
    - Set `PKG_CONFIG_PATH=/opt/mellanox/doca/infrastructure/lib/pkgconfig/:$PKG_CONFIG_PATH`.
    - Verify with `pkg-config --list-all | grep -i doca`.
    - If using DPDK/SPDK/FlexIO, add their respective `.pc` paths (e.g., `/opt/mellanox/dpdk/lib/<arch>-linux-gnu/pkgconfig/`).
- **Phase 3: MPI Discovery**:
    - Check `command -v mpicc`. If empty, search `/usr/mpi/gcc/openmpi-*/bin/mpicc`.
    - Recommend `export PATH=<found-prefix>:$PATH`.
- **Phase 4: Runtime Resource Reservation**:
    - **Hugepages**: 
        1. Calculate `required_bytes / 2 MiB` based on app requirements.
        2. Apply: `echo '<count>' | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages`.
        3. Mount: `sudo mount -t hugetlbfs -o pagesize=2M nodev /mnt/huge`.
    - **Visibility**: Verify `devlink dev show` and `cat /sys/class/net/*/phys_port_name`.
- **Phase 5: Trace Flavor**: If using trace builds, set `LD_LIBRARY_PATH=/opt/mellanox/doca/lib/<arch>-linux-gnu/trace:$LD_LIBRARY_PATH`.

### 3. `test` (Iterative Health Loop)
**Goal**: Verify the environment is healthy before program work.

- **L1: Install Layer**: `pkg-config --modversion doca-common` $\rightarrow$ returns version string.
- **L2: Capability Layer**: `doca_caps --version` $\rightarrow$ matches L1.
- **L3: Smoke Probe**: Build and run one known-good shipped sample.
- **L4: Traffic Loop**: 
    1. Run without traffic (Baseline).
    2. Introduce traffic $\rightarrow$ if fails here, it is **program-class** (route to `doca-programming-guide`).

### 4. `debug` (Env-Class Ladder)
**Goal**: Top-down diagnosis to prevent premature code-fix recommendations.

- **Layer 1: Install**: Is `pkg-config` working? Is the install reachable?
- **Layer 2: Version**: Do `pkg-config` and `doca_caps` match? (Check for partial upgrades).
- **Layer 3: Build**: Check `/tmp/build` vs in-tree. Check build flavor (release vs trace).
- **Layer 4: Runtime**: Hugepages mounted? Modules (`mlx5_core`) loaded? Representors visible?
- **Layer 5: Program**: If L1-L4 are clean $\rightarrow$ route to `doca-programming-guide ## debug`.

### 5. `no-install` (The Stage-1 Fallback)
**Goal**: Guide users without hardware or installs to a productive starting point.

- **The Roadmap**: 
    - **Stage 1 (Container Learning)**: Read API, build samples in NGC container. **No real packets.**
    - **Stage 2 (Hardware Runtime)**: Run against real NIC/DPU.
- **Path 0: NGC DOCA Container (Universal Default)**:
    - **Tag Selection**: Visit `catalog.ngc.nvidia.com`. Match `Arch` $\rightarrow$ `OS` $\rightarrow$ `CUDA` $\rightarrow$ `Flavor`. Pick the highest version.
    - **Execution**: `docker pull nvcr.io/nvidia/doca/doca:<tag>` $\rightarrow$ `docker run -it --rm -v $HOME/dev:/work ... bash`.
- **Other Paths**:
    - **Path A**: Existing Linux + DOCA host (SSH/Remote).
    - **Path B**: Fresh Linux, No NIC (Build-only).
    - **Path C**: Linux + ConnectX/BlueField Hardware (Full Runtime).
- **The Promise**: *"Once inside, paste `pkg-config --modversion doca-<lib>` and I will resume from `doca-programming-guide ## modify`."*

## III. Gotchas

- **The "Container Trap"**: Silently pushing users to containers without running `## recognize` leads to mismatch in deployment expectations.
- **Silent `pkg-config` Failures**: If `PKG_CONFIG_PATH` is not set, `pkg-config` returns empty, which is often mistaken for a missing install.
- **Hugepage Exhaustion**: Defaulting to a fixed page count instead of calculating based on the specific application's requirements.
- **Representor Invisibility**: Trying to run Flow/Switching apps while the device is in `legacy` eswitch mode instead of `switchdev`.
- **Version Drift**: Installing new host packages over an old BFB firmware, leading to `DOCA_ERROR_BAD_STATE`.

## IV. Diagnostic Ladder

| Symptom | Likeliest Cause | Verification | Resolution |
| :--- | :--- | :--- | :--- |
| `pkg-config` cannot find `doca-flow` | Missing `.pc` path | `ls /opt/mellanox/doca/infrastructure/lib/pkgconfig/` | `export PKG_CONFIG_PATH=...` |
| "no free 2048 kB hugepages" | No reservation | `cat /proc/meminfo \| grep Huge` | `echo <count> > .../nr_hugepages` |
| Representors not found in `/sys/class/net` | Wrong eswitch mode | `devlink dev show` | Set mode to `switchdev` (requires consent) |
| `doca_caps` version $\neq$ `pkg-config` version | Partial upgrade | Compare output of both commands | Reinstall consistent DOCA version |
| `mpicc` not found | Non-standard MPI prefix | `ls /usr/mpi/gcc/openmpi-*/bin/mpicc` | `export PATH=<found-prefix>:$PATH` |
