---
name: holoscan-install-debian
description: Native installation of Holoscan SDK C++ runtime and headers on Ubuntu via NVIDIA's apt repository.-
version: "1.0.0"
category: devops
---

# Holoscan Debian/apt Installation

The `holoscan-install-debian` skill enables the native installation of the Holoscan SDK C++ runtime and headers on Ubuntu. This method is preferred for developers building native C++ applications or those deploying on IGX/Jetson platforms where containerization may be suboptimal.

## Invariants

- **C++ Only Scope**: Since Holoscan v3.0.0, the Debian package provides **C++ binaries and headers only**. Python bindings are NOT included. Any Python requirement MUST be handled via the `/holoscan-install-wheel` skill.
- **Ubuntu Exclusive**: This method is strictly limited to Ubuntu (x86_64 22.04/24.04 or ARM64 Jetson/IGX). Other distributions MUST use the Container or Wheel methods.
- **Driver-Package Coupling**: The selected package variant (`holoscan-cuda-12` vs `holoscan-cuda-13`) is strictly bound to the host's NVIDIA driver. A mismatch results in a `"CUDA driver version is insufficient"` runtime error.
- **Live-Doc Authority**: Package names and `cuda-keyring` URLs are volatile. The Debian section of `docs.nvidia.com/holoscan/sdk-user-guide/sdk_installation.html` is the absolute source of truth.
- **Stack Limit Enforcement**: All Holoscan examples MUST be executed with `ulimit -s 32768` to prevent immediate segmentation faults.

## Execution Pipeline

### 1. Environment Audit & Variant Resolution
**Goal**: Determine the correct package variant based on the host driver.
- **Baseline Check**: Run `lsb_release -a` (OS version) and `nvidia-smi` (CUDA driver version).
- **Variant Mapping**:
    - $\text{CUDA} \ge 13.x \rightarrow$ `holoscan-cuda-13`
    - $\text{CUDA} \ge 12.x$ (on IGX) $\rightarrow$ `holoscan`
    - $\text{CUDA} \ge 12.x$ (not on IGX) $\rightarrow$ `holoscan-cuda-12`
    - $\text{CUDA} \ge 12.x$ (nvgpu) $\rightarrow$ `holoscan-cuda-12`

### 2. Repository Configuration
**Goal**: Configure the NVIDIA package manager to recognize Holoscan repositories.
- **Keyring Installation**: Verify if `cuda-keyring` is installed. If missing, fetch the appropriate `.deb` for the Ubuntu version (e.g., `ubuntu2404`) from `developer.download.nvidia.com` and install via `dpkg -i`.
- **Repo Update**: Execute `sudo apt-get update` to refresh the package cache.

### 3. Package Deployment
**Goal**: Install the selected Holoscan SDK variant.
- **Installation**: `sudo apt-get install -y <resolved-package-variant>`.
- **Verification**: Check installation status via `dpkg -l | grep holoscan`.

### 4. Multi-Stage Runtime Verification
**Goal**: Prove the native C++ runtime is correctly linked and functional.
- **Environment Setup**: Set `LD_LIBRARY_PATH=/opt/nvidia/holoscan/lib`, `HOLOSCAN_INPUT_PATH=/opt/nvidia/holoscan/data`, and `ulimit -s 32768`.
- **The Verification Suite**:
    1. **Hello World**: Run `/opt/nvidia/holoscan/examples/hello_world/cpp/hello_world` $\rightarrow$ Expect `"Hello World!"`.
    2. **Tensor Interop**: Run `/opt/nvidia/holoscan/examples/tensor_interop/cpp/tensor_interop` $\rightarrow$ Expect `"Graph execution finished."`.
    3. **Video Replayer (Headless)**: 
        - Ensure data is present: `sudo /opt/nvidia/holoscan/examples/download_example_data`.
        - Patch YAML to `headless: true` (count: 10, repeat: false, realtime: false).
        - Run `/opt/nvidia/holoscan/examples/video_replayer/cpp/video_replayer` $\rightarrow$ Expect Vulkan NVIDIA GPU selection and successful execution.

### 5. Environment Persistence
**Goal**: Provide the user with a reusable configuration snippet for future sessions.
- **Snippet Generation**:
    ```bash
    export LD_LIBRARY_PATH=/opt/nvidia/holoscan/lib:${LD_LIBRARY_PATH}
    export HOLOSCAN_INPUT_PATH=/opt/nvidia/holoscan/data
    ulimit -s 32768
    ```
- **Persistence Advice**: Recommend adding this snippet to `~/.bashrc` or `~/.zshrc`.

## Gotchas

- **The "Missing Python" Surprise**: Users often assume `apt install` provides Python bindings. Explicitly warn that `import holoscan` will fail until `/holoscan-install-wheel` is executed.
- **The Driver Ceiling Crash**: If a user installs `holoscan-cuda-13` on a driver that only supports 12.x, they will see `"CUDA driver version is insufficient"`. The only fix is to swap variants: `sudo apt-get remove -y holoscan-cuda-13 && sudo apt-get install -y holoscan-cuda-12`.
- **Sudo-Induced Env Loss**: Running examples with `sudo` often wipes the `LD_LIBRARY_PATH`. Use `sudo LD_LIBRARY_PATH=/opt/nvidia/holoscan/lib ...` or use a non-privileged user.
- **Shared Library Hell**: `error while loading shared libraries: libholoscan_core.so` is a direct signal that `LD_LIBRARY_PATH` is not set.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `Unable to locate package` | Keyring/Repo missing | `dpkg -l | grep cuda-keyring` | Install `cuda-keyring` $\rightarrow$ `apt-get update` |
| `CUDA driver version insufficient` | Variant mismatch | `nvidia-smi` $\rightarrow$ check CUDA version | Swap `holoscan-cuda-13` $\leftrightarrow$ `holoscan-cuda-12` |
| `import holoscan` fails | Python bindings missing | `pip show holoscan` | Execute `/holoscan-install-wheel` |
| `Segmentation fault` | Stack limit too low | `ulimit -s` | Execute `ulimit -s 32768` |
| `libholoscan_core.so` not found | `LD_LIBRARY_PATH` missing | `echo $LD_LIBRARY_PATH` | Export `/opt/nvidia/holoscan/lib` |
| `video_replayer` data 404 | Dataset not downloaded | `ls /opt/nvidia/holoscan/data` | `sudo /opt/nvidia/holoscan/examples/download_example_data` |

## Command Appendix

| Resource | Purpose | Link |
| :--- | :--- | :--- |
| Installation Guide | Canonical SSoT | [SDK Installation](https://docs.nvidia.com/holoscan/sdk-user-guide/sdk_installation.html) |
| Package Repo | NVIDIA Compute Repos | [developer.download.nvidia.com](https://developer.download.nvidia.com/compute/cuda/repos/) |
| Driver Check | Hardware Baseline | `nvidia-smi` |
| OS Check | Platform Baseline | `lsb_release -a` |
