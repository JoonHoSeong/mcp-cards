---
name: holoscan-install-conda
version: "1.0.0"
description: "Install Holoscan SDK v4.3+ via Conda in a CUDA 13 environment. Use for Conda installs; redirect CUDA 12 hosts to container/wheel."
license: Apache-2.0
metadata:
  author: "Holoscan Team <holoscan-team@nvidia.com>"
  github-url: "https://github.com/nvidia-holoscan/holoscan-sdk"
  tags:
    - holoscan
    - install
    - conda
    - cuda
---

# Holoscan SDK Conda Installation

This skill provides the high-fidelity workflow for installing the Holoscan SDK into a Conda environment. This method is specifically tailored for CUDA 13 hosts and supports both Python runtime and C++ development.

## 📌 Invariants

- **CUDA Version Locked**: This method is strictly for **CUDA 13**. Hosts with CUDA 12 drivers must use `holoscan-install-wheel` or `holoscan-install-container`.
- **OS/Architecture**: Limited to **Linux x86_64**. No support for aarch64 or iGPU via conda-forge.
- **Channel Priority**: The channel order MUST be `-c rapidsai -c conda-forge`. Reversing this order can cause the solver to select outdated `holoscan` builds.
- **Dependency Pinning**: The `cuda-version=13` metapackage must be explicitly pinned to ensure compatible runtime library selection.
- **Stack Size Constraint**: A shell stack size of `32768` is recommended to prevent random segmentation faults during graph execution.

## 🚀 Execution Pipeline

### 1. Environment Prerequisites
Ensure the system meets the minimum requirements and has a functional Conda distribution.

```bash
# Verify CUDA version
nvidia-smi 2>&1 | head -5
# Expected: CUDA Version: 13.x
```

If `conda` is missing, install Miniforge (preferred for conda-forge compatibility):
```bash
wget -q https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh -O /tmp/Miniforge3.sh
bash /tmp/Miniforge3.sh -b -p ~/miniforge3
source ~/miniforge3/etc/profile.d/conda.sh
```

### 2. Conda Environment Provisioning
Create a clean environment with a compatible Python version.

```bash
source ~/miniforge3/etc/profile.d/conda.sh
conda create -n holoscan python=3.13 -y
conda activate holoscan
```

### 3. SDK Installation
Select the package set based on the intended use case.

| Use Case | Required Packages |
| :--- | :--- |
| **Python Runtime Only** | `holoscan` |
| **C++ Development** | `libholoscan-dev` |
| **Combined (Python + C++)** | `holoscan libholoscan-dev` |

Execute the installation with the required runtime dependencies and channel priority:
```bash
# Example: Combined installation
conda install holoscan libholoscan-dev rmm ucxx cuda-version=13 -c rapidsai -c conda-forge -y
```

For C++ development, also install the required build toolchain:
```bash
conda install -c conda-forge cxx-compiler cmake ninja -y
```

### 4. Technical Verification
Verify that the SDK and development headers are correctly mapped.

#### A. Python Binding Check
```bash
python3 -c "import holoscan; print(holoscan.__version__)"
# Expected: Version string (e.g., "4.3.0")
```

#### B. C++ Header Check
```bash
ls "$CONDA_PREFIX/include/holoscan"
# Expected: Directory exists and contains Holoscan headers
```

### 5. Functional Testing
Verify the installation with a graduated test sequence.

```bash
ulimit -s 32768
SDK_VER=$(python3 -c "import holoscan; print(holoscan.__version__)")
BASE="https://raw.githubusercontent.com/nvidia-holoscan/holoscan-sdk/v${SDK_VER}/examples"

# Download examples
curl -fsSL "${BASE}/hello_world/python/hello_world.py" -o /tmp/hs_hello_world.py
curl -fsSL "${BASE}/video_replayer/python/video_replayer.py" -o /tmp/hs_video_replayer.py
curl -fsSL "${BASE}/video_replayer/python/video_replayer.yaml" -o /tmp/video_replayer.yaml

# Patch YAML for headless, 10-frame execution
python3 -c "
c = open('/tmp/video_replayer.yaml').read()
c = c.replace('count: 0', 'count: 10').replace('repeat: true', 'repeat: false').replace('realtime: true', 'realtime: false')
c = c.replace('  width: 854', '  headless: true\\n  width: 854')
open('/tmp/video_replayer.yaml', 'w').write(c)
"

# Run Tests
python3 /tmp/hs_hello_world.py
# Expected: "Hello World!"

HOLOSCAN_INPUT_PATH=/path/to/holoscan/data python3 /tmp/hs_video_replayer.py --config /tmp/video_replayer.yaml
# Expected: "Graph execution finished."
```

## ⚠️ Gotchas

- **The `rmm` Gap**: `rmm` (RAPIDS Memory Manager) is a runtime dependency of `holoscan` but is not always explicitly declared in the metadata. If `import holoscan` fails with `librmm.so` missing, you must install `rmm` manually.
- **Channel Order Trap**: Using `-c conda-forge -c rapidsai` (wrong order) often leads the solver to a legacy `holoscan` build from conda-forge instead of the latest from rapidsai.
- **C++ Config Failures**: `find_package(holoscan)` in CMake will fail if only `holoscan` (Python) is installed. `libholoscan-dev` is required for the CMake configuration and headers.
- **Silent BashRC Exclusion**: Installing Miniforge with `-b` does not modify `~/.bashrc`. Users will see `conda: command not found` in new shells unless they manually source the profile script.

## 🛠️ Diagnostic Ladder

| Symptom | Probable Cause | Verification | Fix |
| :--- | :--- | :--- | :--- |
| `ImportError: librmm.so` | `rmm` missing from environment | `conda list \| grep rmm` | `conda install rmm -c rapidsai` |
| Installed version $\neq$ Official version | Incorrect channel priority | Check `conda list` and channel source | Re-install with `-c rapidsai -c conda-forge` |
| Segfault on app startup | Default shell stack size too low | `ulimit -s` (check if < 32768) | `ulimit -s 32768` |
| `find_package(holoscan)` fails | `libholoscan-dev` missing | `ls $CONDA_PREFIX/include/holoscan` | `conda install libholoscan-dev -c conda-forge` |
| `conda: command not found` | Miniforge not sourced in shell | `ls ~/miniforge3/bin/conda` | `source ~/miniforge3/etc/profile.d/conda.sh` |
