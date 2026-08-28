---
name: holoscan-install-wheel
version: "1.0.0"
description: "Install Holoscan SDK Python wheel via pip into a venv. Use for Python installs; not for native C++/apt or Conda installs."
license: Apache-2.0
metadata:
  author: "Holoscan Team <holoscan-team@nvidia.com>"
  github-url: "https://github.com/nvidia-holoscan/holoscan-sdk"
  tags:
    - holoscan
    - install
    - pip
    - wheel
    - python
---

# Holoscan SDK Python Wheel Installation

This skill provides the high-fidelity workflow for installing the Holoscan SDK Python bindings using pip wheels. This method is intended for Python-centric development and is separate from the native C++/apt installation.

## 📌 Invariants

- **Virtual Environment Mandatory**: Direct system-wide pip installation is prohibited, especially on Ubuntu 24.04 (PEP 668), which blocks system pip entirely.
- **CUDA Major Alignment**: The pip package name MUST match the host CUDA major version.
    - Host CUDA 12.x $\rightarrow$ `holoscan-cu12`
    - Host CUDA 13.x $\rightarrow$ `holoscan-cu13`
- **Python Version Support**: Python 3.10 through 3.13.
- **Stack Size Constraint**: A shell stack size of `32768` is required to prevent `RuntimeWarning` and potential segmentation faults during graph execution.
- **Dependency Chain**: Python-only installation provides bindings. For full C++ headers and system libraries, this must be paired with `holoscan-install-debian`.

## 🚀 Execution Pipeline

### 1. CUDA Variant Determination
Identify the host CUDA version to select the correct wheel.

```bash
# Check driver-reported CUDA version
nvidia-smi 2>&1 | head -5
```

| Driver CUDA Version | Required Pip Package |
| :--- | :--- |
| 13.x+ | `holoscan-cu13` |
| 12.x | `holoscan-cu12` |

### 2. Environment Provisioning
Create and activate a dedicated virtual environment to avoid package conflicts.

```bash
# Create venv if it doesn't exist
python3 -m venv ~/holoscan/venv

# Activate the environment
source ~/holoscan/venv/bin/activate
```

### 3. SDK Installation
Install the matched CUDA variant wheel via pip.

```bash
# Replace <cu12|cu13> with the variant determined in Step 1
pip install holoscan-cu12   # Example for CUDA 12.x
```

### 4. Technical Verification
Verify the installation through a graduated testing sequence. Ensure the venv is active.

#### A. Basic Import & Version Check
```bash
python3 -c "import holoscan; print(holoscan.__version__)"
# Expected: Version string (e.g., "4.1.0")
```

#### B. Functional Test: Hello World
Fetch the official example and execute with the required stack limit.

```bash
SDK_VER=$(python3 -c "import holoscan; print(holoscan.__version__)")
BASE="https://raw.githubusercontent.com/nvidia-holoscan/holoscan-sdk/v${SDK_VER}/examples"

# Download and run
curl -fsSL "${BASE}/hello_world/python/hello_world.py" -o /tmp/hs_hello_world.py
ulimit -s 32768 && python3 /tmp/hs_hello_world.py
# Expected: "Hello World!"
```

#### C. Integration Test: Video Replayer (Headless)
Verify graph execution and data path handling.

```bash
# 1. Download assets
curl -fsSL "${BASE}/video_replayer/python/video_replayer.py" -o /tmp/hs_video_replayer.py
curl -fsSL "${BASE}/video_replayer/python/video_replayer.yaml" -o /tmp/hs_video_replayer.yaml

# 2. Configure for headless execution (10 frames, no GUI)
python3 -c "
c = open('/tmp/hs_video_replayer.yaml').read()
c = c.replace('count: 0','count: 10').replace('repeat: true','repeat: false').replace('realtime: true','realtime: false')
c = c.replace('holoviz:\\n  width: 854','holoviz:\\n  headless: true\\n  width: 854')
open('/tmp/hs_video_replayer_run.yaml','w').write(c)
"

# 3. Execute with data path
# Note: /opt/nvidia/holoscan/data is provided by the Debian package
ulimit -s 32768 && HOLOSCAN_INPUT_PATH=/opt/nvidia/holoscan/data \
  python3 /tmp/hs_video_replayer.py --config /tmp/hs_video_replayer_run.yaml
# Expected: "Graph execution finished."
```

## ⚠️ Gotchas

- **Ubuntu 24.04 Block**: Attempting `pip install` outside a venv results in `error: externally-managed-environment`. Always use `venv`.
- **Silent Segfaults**: If `ulimit -s 32768` is omitted, the application may crash without a clear error message during heavy graph processing.
- **Data Missing for Replayer**: The `video_replayer` requires the `racerx/` dataset. This data is typically installed via the Debian package. If not present, the user must manually set `HOLOSCAN_INPUT_PATH` to the directory containing the dataset.
- **Wheel Mismatch**: Installing `holoscan-cu13` on a CUDA 12 driver will result in `ImportError` or `CUDA_ERROR_INVALID_DEVICE`.

## 🛠️ Diagnostic Ladder

| Symptom | Probable Cause | Verification | Fix |
| :--- | :--- | :--- | :--- |
| `externally-managed-environment` | Ubuntu 24.04 system pip restriction | Check OS version $\rightarrow$ Check if venv is active | Create and `source` a python venv |
| `ImportError` or CUDA error at `import holoscan` | Wheel variant $\neq$ Host CUDA major | Compare `nvidia-smi` output with `pip list` | `pip uninstall holoscan-cuXX` $\rightarrow$ install matching variant |
| `RuntimeWarning: stack size` or Segfault | Default shell stack size too low | Run `ulimit -s` (default is often 8192) | Run `ulimit -s 32768` in current shell |
| `video_replayer` cannot find `racerx/` | `HOLOSCAN_INPUT_PATH` invalid or data missing | `ls $HOLOSCAN_INPUT_PATH/racerx` | Install Debian package or set `HOLOSCAN_INPUT_PATH` to correct data root |
