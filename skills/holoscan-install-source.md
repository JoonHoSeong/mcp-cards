---
name: holoscan-install-source
version: "1.0.0"
description: "Build Holoscan SDK from source via the in-tree ./run script. Use only when published packages don't meet the user's needs."
license: Apache-2.0
metadata:
  author: "Holoscan Team <holoscan-team@nvidia.com>"
  github-url: "https://github.com/nvidia-holoscan/holoscan-sdk"
  tags:
    - holoscan
    - install
    - source
    - build
    - cmake
---

# Holoscan SDK — Build from Source

This skill provides the high-fidelity workflow for building the Holoscan SDK from the official source tree. This process utilizes a containerized build system via the `./run` script to ensure environment consistency while producing a local install tree for use as a CMake dependency.

## 📌 Invariants

- **Use Case Restriction**: Source builds are strictly reserved for scenarios where published packages (Conda, apt, wheel, container) are insufficient—specifically for debugging (symbols), custom CMake configurations, or unsupported hardware/software combinations.
- **Container Dependency**: This is not a bare-metal build. The `./run` script mandates a functional Docker environment with the NVIDIA Container Toolkit.
- **Resource Requirements**: 
    - **Disk Space**: Minimum ~20 GB free for the build container and resulting install trees.
    - **Time**: A clean initial build typically requires 10–30 minutes.
- **Cross-Compilation**: Builds for `aarch64` on x86_64 hosts require `qemu-user-static` to be installed on the host OS.

## 🚀 Execution Pipeline

### 1. Host Environment Verification
Ensure the host has the necessary orchestration tools and GPU passthrough capabilities.

```bash
# Verify core tools
git --version
docker --version

# Verify NVIDIA Container Toolkit (GPU passthrough)
docker run --rm --gpus all ubuntu:22.04 nvidia-smi
```

If GPU passthrough fails, install the NVIDIA Container Toolkit:
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker
```

### 2. Source Acquisition
Clone the repository and check out a stable release tag.

```bash
mkdir -p ~/holoscan/
git clone https://github.com/nvidia-holoscan/holoscan-sdk.git
cd ~/holoscan/holoscan-sdk

# List recent tags to find a stable version
git tag | grep -E '^v[0-9]' | sort -V | tail -5
# Checkout a specific version (e.g., v4.1.0)
git checkout v<VERSION>
```

### 3. Containerized Build
Execute the build process. The `./run build` script handles the container lifecycle, CMake configuration, and installation.

```bash
# Standard build
./run build
```

**Build Configuration Options:**
| Flag | Purpose |
| :--- | :--- |
| `--type debug` | Build with debug symbols and no optimization |
| `--type RelWithDebInfo` | Release build with debug symbols |
| `--arch aarch64` | Cross-compile for ARM64 (requires `qemu-user-static`) |
| `--gpu igpu` | Build for Jetson/IGX iGPUs |
| `--dryrun` | Preview commands without executing |

*Note: If changing options, clear the cache first: `./run clear_cache && ./run build`.*

### 4. Technical Verification
Run a comprehensive test suite to verify the integrity of the build.

```bash
# Execute core verification tests (regex must be single-quoted to avoid bash pipe interpretation)
./run test --options "-R 'EXAMPLE_CPP_HELLO_WORLD_TEST|EXAMPLE_PYTHON_HELLO_WORLD_TEST|EXAMPLE_CPP_TENSOR_INTEROP_TEST|EXAMPLE_PYTHON_TENSOR_INTEROP_TEST|EXAMPLE_CPP_VIDEO_REPLAYER_TEST|EXAMPLE_PYTHON_VIDEO_REPLAYER_TEST' --output-on-failure"
```

**Expected Result**: All six tests must pass. Any failure indicates a build corruption or environment mismatch.

### 5. Integration with External Applications
The build produces an install tree that can be used as a `CMAKE_PREFIX_PATH` for other projects.

**Install Tree Path**: `/path/to/holoscan-sdk/install-cu<N>-<arch>/`

Set this path via `Holoscan_ROOT` or `CMAKE_PREFIX_PATH` when building dependent applications.

## ⚠️ Gotchas

- **Bash Pipe Interpretation**: If the test regex (containing `|`) is not wrapped in single quotes, bash will interpret it as a pipe, leading to a `command not found` error.
- **Buildx Missing**: Some environments lack the `docker-buildx-plugin` required for advanced builds. Fix: `sudo apt-get install docker-buildx-plugin`.
- **Cross-Compile Failures**: aarch64 builds will fail silently or with obscure errors if `qemu-user-static` is not installed on the host.
- **Stale CMake Cache**: Changing build types (e.g., Release $\rightarrow$ Debug) without running `./run clear_cache` often leads to inconsistent binaries.

## 🛠️ Diagnostic Ladder

| Symptom | Probable Cause | Verification | Fix |
| :--- | :--- | :--- | :--- |
| `bash: <TEST_NAME>: command not found` | Unquoted regex in `./run test` | Check command for `|` without single quotes | Wrap regex in single quotes: `--options "-R '<regex>'"` |
| GPU not visible in container | NVIDIA Container Toolkit misconfigured | `docker run --rm --gpus all ubuntu:22.04 nvidia-smi` | Run `sudo nvidia-ctk runtime configure --runtime=docker` $\rightarrow$ restart docker |
| CMake config errors after flag change | Stale CMake cache in build dir | Check for existing `CMakeCache.txt` | `./run clear_cache && ./run build` |
| Cross-compilation (aarch64) fails | Missing QEMU emulation | `dpkg -l \| grep qemu-user-static` | `sudo apt-get install qemu-user-static` |
| `docker-buildx` not found | Missing buildx plugin | `docker buildx version` | `sudo apt-get install docker-buildx-plugin` |
