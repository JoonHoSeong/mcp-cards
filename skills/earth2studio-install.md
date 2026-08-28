---
name: earth2studio-install
description: High-precision guide for installing Earth2Studio, managing optional model extras, and configuring the runtime environment.
version: 0.16.0
category: devops
---

# Earth2Studio Installation

The `earth2studio-install` skill provides a deterministic workflow for deploying Earth2Studio. It focuses on the precise selection of optional extras (models, submodules) and the configuration of environment variables to ensure a stable, high-performance weather/climate AI environment.

## Invariants

- **Zero-Auto-Install**: The agent MUST NEVER execute installation commands (`pip install`, `uv add`, etc.) directly on the user's system. All commands must be provided as text for the user to execute.
- **Live-Doc Authority**: Installation commands, version tags, and extra names are volatile. All recommendations MUST be preceded by a fetch of the current live installation docs (`nvidia.github.io/earth2studio/userguide/about/install.html`).
- **Verification-First**: An installation is not considered "complete" until the user verifies the version via `import earth2studio; earth2studio.__version__`.
- **Python Version Guard**: Earth2Studio requires Python 3.10+. The current recommended version (e.g., 3.13) must be enforced to avoid dependency conflicts.

## Execution Pipeline

### 1. Live Doc Synchronization
**Goal**: Obtain the current state of the package registry.
- **Fetch**: Access the official installation page.
- **Extract**:
    - Current release version tag (e.g., `@0.16.0`).
    - Full list of available optional extras (Prognostic, Diagnostic, DA, Submodules).
    - Specific build flags (e.g., `--no-build-isolation` for pip).

### 2. Environment Triage
**Goal**: Align the installation method with the user's project structure.
- **Tool Selection**: 
    - **`uv` (Recommended)**: Use for faster installs and better handling of URL-based transitive dependencies.
    - **`pip`**: Use for standard PyPI installations (requires more manual pre-install steps).
- **Context Identification**: New project vs. adding to an existing environment.

### 3. Base Installation & Verification
**Goal**: Establish the core library foundation.
- **Command Generation**: Provide the exact `uv` or `pip` command based on the live docs.
- **Confirmation Loop**: Wait for the user to execute the command and provide the output of `earth2studio.__version__`.

### 4. Optional Extra Selection
**Goal**: Minimize environment bloat while providing required model capabilities.
- **Category-Based Offering**: Present extras grouped by use case:
    - **Prognostic**: `aifs`, `graphcast`, `pangu`, `sfno`, etc.
    - **Diagnostic**: `corrdiff`, `climatenet`, etc.
    - **Data Assimilation**: `da-healda`, `da-stormcast`, etc.
    - **Submodules**: `data`, `perturbation`, `statistics`.
- **Optimization**: Suggest `--extra all` only for `uv` users who require the full suite.

### 5. Specialized Dependency Deployment
**Goal**: Handle complex builds and hardware-specific requirements.
- **Build-Time Warnings**: Explicitly warn about long build times (10-30+ mins) for:
    - `flash-attention` (AIFS variants).
    - `natten` (Atlas, StormScope).
    - `torch-harmonics` (FCN3, SFNO).
- **Manual Pre-installs**: For `pip` users, provide the necessary pre-install sequence for `earth2grid`, `torch-harmonics`, or `makani`.
- **DA Requirements**: Ensure CUDA 12 + CuPy + cuDF are present for Data Assimilation models.

### 6. Environment Configuration (Optional)
**Goal**: Optimize cache and timeout settings for large model downloads.
- **Variable Tuning**:
    - `EARTH2STUDIO_CACHE`: Set for custom general cache paths.
    - `EARTH2STUDIO_DATA_CACHE` / `EARTH2STUDIO_MODEL_CACHE`: Divert data and weights to high-capacity drives.
    - `EARTH2STUDIO_PACKAGE_TIMEOUT`: Increase for unstable network environments.

## Gotchas

- **The `pip` Build Isolation Trap**: Some complex CUDA extensions fail if build isolation is enabled. Always suggest `--no-build-isolation` when installing `torch-harmonics` or `natten` via pip.
- **CUDA Toolkit Mismatch**: `flash-attention` builds often fail if the system CUDA toolkit version differs from the PyTorch CUDA version.
- **Missing System Headers**: `Python.h` errors indicate missing `python3-dev` headers; `eccodes` errors indicate missing `libeccodes-dev`.
- **The "Silent Weight Download"**: Installation only installs the *code*. Model weights are downloaded lazily during the first inference call, which can lead to unexpected delays or disk-full errors at runtime.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `ModuleNotFoundError` | Missing extra | `pip list` $\rightarrow$ check for extra name | Install specific extra (e.g., `pip install earth2studio[graphcast]`) |
| Build failure in `natten` | CUDA/PyTorch mismatch | `nvcc --version` vs `torch.version.cuda` | Align CUDA Toolkit version with PyTorch |
| `ecCodes` missing | System lib missing | `ldconfig -p | grep eccodes` | `sudo apt-get install libeccodes-dev` |
| `Python.h` not found | Missing dev headers | `ls /usr/include/python3.x` | `sudo apt-get install python3-dev` |
| `torch.cuda.is_available() == False` | Driver/Toolkit mismatch | `nvidia-smi` $\rightarrow$ check driver version | Update NVIDIA Driver or PyTorch CUDA version |

## Command Appendix

| Resource | Purpose | Link |
| :--- | :--- | :--- |
| Installation Docs | Live SSoT for commands | [Install Guide](https://nvidia.github.io/earth2studio/userguide/about/install.html) |
| Troubleshooting | Official Error Guide | [Troubleshooting](https://nvidia.github.io/earth2studio/userguide/support/troubleshooting.html) |
| FAQ | Common Questions | [FAQ](https://nvidia.github.io/earth2studio/userguide/support/faq.html) |
| uv Installation | Recommended manager | [uv Docs](https://docs.astral.sh/uv/getting-started/installation/) |
