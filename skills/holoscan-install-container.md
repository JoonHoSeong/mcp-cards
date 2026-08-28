---
name: holoscan-install-container
version: "1.0.0"
description: "Install Holoscan SDK via the NGC Docker container. Use for container-based installs; not for native apt/pip/Conda installs."
license: Apache-2.0
metadata:
  author: "Holoscan Team <holoscan-team@nvidia.com>"
  github-url: "https://github.com/nvidia-holoscan/holoscan-sdk"
  tags:
    - holoscan
    - install
    - container
    - docker
    - ngc
---

# Holoscan NGC Container Installation

This skill provides the high-fidelity workflow for deploying the Holoscan SDK using official NGC Docker containers. This method ensures a fully pre-configured environment, bypassing host-level dependency conflicts.

## 📌 Invariants

- **GPU Passthrough Mandatory**: The host must have the NVIDIA Container Toolkit installed and configured to allow the container to access the GPU.
- **Tag-to-Hardware Alignment**: The container tag suffix MUST match the host GPU and CUDA driver version.
    - CUDA 13.x+ $\rightarrow$ `cuda13`
    - CUDA 12.x (dGPU/Ampere/Ada) $\rightarrow$ `cuda12-dgpu`
    - CUDA 12.x (ARM64/iGPU/nvgpu) $\rightarrow$ `cuda12-igpu`
- **Resource Requirements**: 10–20 GB of free disk space for image pulls.
- **Runtime Constraints**: 
    - **Stack Size**: `ulimit -s 32768` must be applied *inside* the container shell before executing Holoscan applications.
    - **GUI/X11**: GUI examples require X11 forwarding. For SSH or headless environments, YAML configurations must be patched to `headless: true`.

## 🚀 Execution Pipeline

### 1. Tag Selection
Determine the correct image tag based on the host's `nvidia-smi` output.

```bash
nvidia-smi 2>&1 | head -5
```

| `nvidia-smi` CUDA Version | Required Tag Suffix |
| :--- | :--- |
| 13.x+ | `cuda13` |
| 12.x (dGPU) | `cuda12-dgpu` |
| 12.x (iGPU/ARM64) | `cuda12-igpu` |

**Final Tag Format**: `<version>-<suffix>` (e.g., `v4.1.0-cuda13`).

### 2. Host Verification & Image Pull
Ensure the NVIDIA runtime is operational and pull the image.

```bash
# Verify GPU passthrough
docker run --rm --gpus all ubuntu:22.04 nvidia-smi 2>&1 | tail -5

# Pull image (Warn user: ~10-20GB)
docker pull nvcr.io/nvidia/clara-holoscan/holoscan:<TAG>
```

### 3. Graduated Functional Verification
Verify the containerized installation using a sequence of tests. Use the following base run command:

```bash
IMG=nvcr.io/nvidia/clara-holoscan/holoscan:<TAG>
RUN=(docker run --rm --runtime=nvidia --gpus all --cap-add CAP_SYS_PTRACE --ipc=host --ulimit memlock=-1 --ulimit stack=67108864)
```

#### A. Basic Runtime Tests
```bash
# Python hello_world
"${RUN[@]}" "$IMG" bash -c "ulimit -s 32768 && python3 /opt/nvidia/holoscan/examples/hello_world/python/hello_world.py"
# Expected: "Hello World!"

# C++ hello_world
"${RUN[@]}" "$IMG" bash -c "ulimit -s 32768 && /opt/nvidia/holoscan/examples/hello_world/cpp/hello_world"
# Expected: "Hello World!"

# C++ tensor_interop
"${RUN[@]}" "$IMG" bash -c "ulimit -s 32768 && /opt/nvidia/holoscan/examples/tensor_interop/cpp/tensor_interop"
# Expected: "Graph execution finished."
```

#### B. Holoviz/Vulkan Tests (Headless)
Inject `headless: true` into the YAML to avoid X11 errors.

```bash
# Python tensor_interop (10 frames, headless)
"${RUN[@]}" "$IMG" bash -c "
  ulimit -s 32768
  sed -e 's/count: 0/count: 10/' -e 's/repeat: true/repeat: false/' -e 's/realtime: true/realtime: false/' -e 's/^holoviz:/holoviz:\\n  headless: true/' /opt/nvidia/holoscan/examples/tensor_interop/python/tensor_interop.yaml > /tmp/ti.yaml
  cd /opt/nvidia/holoscan/examples/tensor_interop/python
  python3 tensor_interop.py --config /tmp/ti.yaml
"

# Python video_replayer (10 frames, headless)
"${RUN[@]}" "$IMG" bash -c "
  ulimit -s 32768
  sed -e 's/count: 0/count: 10/' -e 's/repeat: true/repeat: false/' -e 's/realtime: true/realtime: false/' -e 's/^  width: 854/  headless: true\\n  width: 854/' /opt/nvidia/holoscan/examples/video_replayer/python/video_replayer.yaml > /tmp/vr.yaml
  cd /opt/nvidia/holoscan/examples/video_replayer/python
  HOLOSCAN_INPUT_PATH=/opt/nvidia/holoscan/data python3 video_replayer.py --config /tmp/vr.yaml
"

# C++ video_replayer (10 frames, headless)
"${RUN[@]}" "$IMG" bash -c "
  ulimit -s 32768
  sed -e 's/count: 0/count: 10/' -e 's/repeat: true/repeat: false/' -e 's/realtime: true/realtime: false/' -e 's/^  width: 854/  headless: true\\n  width: 854/' /opt/nvidia/holoscan/examples/video_replayer/cpp/video_replayer.yaml > /tmp/vr_cpp.yaml
  cd /opt/nvidia/holoscan/examples/video_replayer/cpp
  HOLOSCAN_INPUT_PATH=/opt/nvidia/holoscan/data ./video_replayer --config /tmp/vr_cpp.yaml
"
```

### 4. Production Launch Command
To start an interactive session with the SDK:

```bash
docker run -it --rm \
  --runtime=nvidia --gpus all --cap-add CAP_SYS_PTRACE \
  --ipc=host --ulimit memlock=-1 --ulimit stack=67108864 \
  nvcr.io/nvidia/clara-holoscan/holoscan:<TAG>
```

## ⚠️ Gotchas

- **Driver Mismatch**: Selecting `cuda12-dgpu` on a `cuda13` host (or vice versa) will lead to CUDA initialization failures. Always align the tag with `nvidia-smi`.
- **X11 Hangs**: Running Holoviz examples over SSH without patching `headless: true` in the YAML often results in the application hanging or crashing due to missing display servers.
- **Missing Data**: The `video_replayer` requires the `racerx` dataset. In the container, this is located at `/opt/nvidia/holoscan/data`. Ensure `HOLOSCAN_INPUT_PATH` is set to this location.
- **Container Segfaults**: Applying `ulimit -s 32768` on the host is NOT enough; it must be applied *inside* the container shell before the application starts.

## 🛠️ Diagnostic Ladder

| Symptom | Probable Cause | Verification | Fix |
| :--- | :--- | :--- | :--- |
| `could not select device driver "nvidia"` | NVIDIA Container Toolkit missing | `docker run --rm --gpus all ubuntu:22.04 nvidia-smi` | Install NVIDIA Container Toolkit $\rightarrow$ Restart Docker |
| CUDA initialization failure | Tag suffix $\neq$ Host GPU/Driver | Compare `nvidia-smi` CUDA version with Image Tag | Pull the correct tag (e.g., `cuda13` vs `cuda12-dgpu`) |
| Segfault on example launch | Stack size too low inside container | Check for `ulimit -s 32768` in the `docker run` command | Use `bash -c "ulimit -s 32768 && ..."` pattern |
| Holoviz example hangs over SSH | GUI attempt in headless env | Check YAML for `headless: true` | Use `sed` to inject `headless: true` under the `holoviz:` section |
| `video_replayer` cannot find data | `HOLOSCAN_INPUT_PATH` not set | `ls /opt/nvidia/holoscan/data/racerx` | Set `HOLOSCAN_INPUT_PATH=/opt/nvidia/holoscan/data` |
