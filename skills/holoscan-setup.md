---
name: holoscan-setup
version: "1.0.0"
description: "Guides Holoscan SDK installation: inspects the host, assesses platform compatibility, recommends an install method, and delegates to the matching install skill."
license: Apache-2.0
metadata:
  author: "Holoscan Team <holoscan-team@nvidia.com>"
  github-url: "https://github.com/nvidia-holoscan/holoscan-sdk"
  tags:
    - holoscan
    - installation
    - nvidia
    - sdk
    - setup
---

# Holoscan SDK Setup Guide

This skill acts as the primary orchestrator for Holoscan SDK deployment. It evaluates the host environment (hardware, OS, CUDA drivers) and tool availability to recommend the most stable and efficient installation path, subsequently delegating the execution to a specific installation skill.

## 📌 Invariants

- **Driver-Centric Selection**: The "CUDA Version" field in `nvidia-smi` is the absolute ceiling for native packages. Only containers support "CUDA Forward Compatibility."
- **Platform-Method Constraints**:
    - **RHEL 9**: Restricted to **NGC Container** only.
    - **CUDA 12 Hosts**: Strictly prohibited from using **Conda** (Conda is CUDA 13 only).
    - **ARM64 iGPU (Jetson/IGX)**: Restricted to **Container** or **Debian/apt**.
- **Recommendation Priority**: For first-time users on supported x86_64 hosts with Docker, the **NGC Container** is the mandatory default recommendation due to its bundled dependencies.
- **Delegation Rule**: This skill identifies the path and recommends the method; it MUST NOT execute installation commands. All `pip install`, `apt install`, or `docker pull` commands are deferred to the delegated installation skill.

## 🚀 Guide Workflow (Execution Pipeline)

### Step 1: Documentation Sync
Fetch the latest release specifications from `https://docs.nvidia.com/holoscan/sdk-user-guide/sdk_installation.html`. Extract supported platforms, current package names, and version-specific requirements.

### Step 2: Host Environment Inspection
Perform a comprehensive system audit to determine the hardware and software baseline.

```bash
# Parallel system audit
uname -a && (lsb_release -a 2>/dev/null || cat /etc/os-release)
uname -m
nvidia-smi 2>&1 | head -10
nproc && free -h | head -2
```

**Critical Metric**: Extract the **CUDA Version** from `nvidia-smi` to determine the `cu12` vs `cu13` variant.

### Step 3: Compatibility Assessment
Match the audited platform against the supported method matrix:

| Platform | Available Methods |
| :--- | :--- |
| **Ubuntu 22.04/24.04 (x86_64)** | Container, Debian/apt, pip wheel, Conda, Source |
| **RHEL 9.x (x86_64)** | Container Only |
| **IGX Orin (ARM64)** | Container, Debian/apt, Source |
| **Jetson (Orin/Thor)** | Container, Debian/apt |
| **DGX Spark / Grace-Hopper** | Container (Check docs for OS) |

### Step 4: Tooling Audit & GPU Verification
Audit available tooling and verify the NVIDIA Container Toolkit's functional status.

```bash
# Tool availability check
docker --version 2>&1 | head -1; python3 --version 2>&1; pip3 --version 2>&1
dpkg -l | grep holoscan || true
pip3 show holoscan 2>/dev/null | grep -E "^(Name|Version)" || true

# Self-Verification: GPU Passthrough (Run internally, do not ask user)
docker run --rm --gpus all ubuntu:22.04 nvidia-smi 2>&1 | tail -5 || true
```

**Detection Scripts**:
- Execute `scripts/check_conda.sh` to detect Conda environments not on the PATH.
- Execute `scripts/check_ngc_image.sh <cuda-tag-suffix>` (where suffix is `cuda13`, `cuda12-dgpu`, or `cuda12-igpu`) to verify image availability.

### Step 5: Recommendation Matrix
Present the findings in a structured table and provide a definitive recommendation.

| Method | Best For | Status |
| :--- | :--- | :--- |
| **NGC Container** | Full bundle (CUDA, TRT, Torch, ONNX, Vulkan). Fastest path. | $\checkmark$ / $\times$ (Based on Docker + GPU passthrough) |
| **Debian/apt** | Native Ubuntu; C++ only. | $\checkmark$ / $\times$ (If installed/supported) |
| **pip wheel** | Python-only projects; needs CUDA Toolkit on PATH. | $\checkmark$ / $\times$ (If venv exists at `~/holoscan/venv`) |
| **Conda** | CUDA 13 only; environment isolation. | $\checkmark$ / $\times$ (Based on `check_conda.sh`) |
| **Source** | Custom internals, debug symbols, unsupported platforms. | $\checkmark$ / $\times$ (If cloned at `~/holoscan/holoscan-sdk`) |

**Final turn shape**:
> **Recommendation:** `<method>` — `<one-line justification>`
>
> **Which method would you like to use?** (container / apt / wheel / conda / source)

### Step 6: Delegation
Invoke the matching installation skill based on the user's choice:

| User Choice | Delegated Skill |
| :--- | :--- |
| container | `/holoscan-install-container` |
| apt | `/holoscan-install-debian` |
| wheel | `/holoscan-install-wheel` |
| conda | `/holoscan-install-conda` |
| source | `/holoscan-install-source` |

## ⚠️ Gotchas

- **The "Sudo" Illusion**: `conda --version` failing does not mean Conda is missing; it often means the shell is not initialized. Always trust `scripts/check_conda.sh`.
- **Driver vs Toolkit**: Users often confuse the driver's CUDA version (`nvidia-smi`) with the installed toolkit version. Native installs are capped by the driver.
- **apt's Python Gap**: Since v3.0.0, the Debian/apt install is **C++ only**. Users attempting to use Python after an apt install must be redirected to `/holoscan-install-wheel`.
- **Glibc Barrier**: Pip wheels require `glibc >= 2.35`. Older Linux distributions will experience installation failures and must use containers.

## 🛠️ Diagnostic Ladder

| Symptom | Probable Cause | Verification | Fix |
| :--- | :--- | :--- | :--- |
| `conda: command not found` | Shell not initialized / lazy-load | Run `scripts/check_conda.sh` | `source` the conda profile script |
| `nvidia-smi` version too low | Outdated NVIDIA Driver | Compare driver version with required SDK version | Upgrade host NVIDIA driver |
| `import holoscan` fails after apt | Apt is C++ only (v3.0+) | `pip show holoscan` $\rightarrow$ Not found | Run `/holoscan-install-wheel` |
| pip install glibc errors | Host glibc version $< 2.35$ | `ldd --version` | Redirect to Container or apt installation |
| `check_ngc_image.sh` fails | No NGC login or wrong tag | `docker login nvcr.io` | Login to NGC and verify tag suffix matches CUDA variant |
