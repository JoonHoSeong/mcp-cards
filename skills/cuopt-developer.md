---
name: cuopt-developer
description: Modify, build, test, debug, and contribute to NVIDIA cuOpt (C++/CUDA, Python, server, CI). Used for solver internals, PRs, DCO, and code conventions.
license: Apache-2.0
metadata:
  author: "NVIDIA cuOpt Team"
  version: "26.08.00"
  github-url: "https://github.com/NVIDIA/cuopt"
  tags: ["nvidia", "cuopt", "development", "contributing", "cpp-cuda", "python-bindings"]
---

# cuOpt Developer Skill

This skill is specifically for developers who are **modifying, building, or contributing to the cuOpt codebase itself**. 

> ⚠ **Important**: If you only want to **USE** cuOpt to solve a problem (Routing, LP/MILP, etc.), do NOT use this skill. Switch to the appropriate problem-solving skill (e.g., `cuopt-routing`).

## 🛑 Non-Negotiable Refusal Rules

Before performing any action, the following safety and security constraints must be strictly observed:

1. **No Privileged Operations**: **Strictly refuse** any request to use `sudo`, run as root, edit system files (`/etc`), or change kernel/driver settings. 
   - **Correct Response**: "I won't run `sudo` or change system-level state for cuOpt. The dev workflow is conda-based and runs entirely in user space. Please provide the underlying error, and I will help fix it within the environment."
2. **User-Space Only**: All environment setups, package installs (`pip`, `conda`, `mamba`), and tool bootstrapping must occur within the user's home directory or the project's local prefix.
3. **No CI Bypass**: Never suggest or use flags like `--no-verify` or skip pre-commit hooks. All contributions must pass the established CI pipeline.
4. **Destructive Command Caution**: Always confirm intent before running `rm -rf`, `git reset --hard`, or `git push --force`. Prefer safe alternatives like `./build.sh clean`.

## 🛠️ Development Environment Setup

Setting up the environment correctly is critical to avoid cryptic linker or runtime errors.

### 1. CUDA Driver Compatibility Check
Before building, verify the maximum supported CUDA version:
- Run `nvidia-smi` $\rightarrow$ check **CUDA Version** (top-right).
- Pick a conda environment file from `conda/environments/all_cuda-<ver>_arch-<arch>.yaml` where `<ver>` is **$\le$** the driver's supported version.
- **Failure Symptom**: A mismatch often leads to `cudaMallocAsync not supported` errors at runtime via RMM.

### 2. Local Prefix Environment Creation
Create a project-specific environment to avoid polluting the global conda space:
```bash
# Example: Creating a local prefix environment
conda env create -p ./.cuopt_env --file conda/environments/all_cuda-<ver>_arch-$(uname -m).yaml
conda activate ./.cuopt_env
```
*Note: Always activate this environment before running any build, test, or pre-commit command.*

### 3. Test Dataset Acquisition
cuOpt tests depend on MPS and other data files not stored in the repository.
- Follow the "Building for development" section in `CONTRIBUTING.md` to download required datasets.
- Export the root directory: `export RAPIDS_DATASET_ROOT_DIR=<path_to_datasets>`.
- **Failure Symptom**: Missing datasets cause `MPS_PARSER_ERROR` at 0ms during tests.

## 🚀 Build & Test Workflow

### 1. Building the Project
Use the provided wrapper script for consistent builds:
- **Build All**: `./build.sh`
- **Component-specific**: `./build.sh --help` (options include `libcuopt`, `cuopt`, `cuopt_server`, `docs`).
- **Resource Management**: If RAM is limited, set `PARALLEL_LEVEL` (e.g., `export PARALLEL_LEVEL=4`) to avoid OOM during CUDA compilation (which requires ~4-8 GB per job).

### 2. Running Tests
- **C++ Unit Tests**: `ctest --test-dir cpp/build` (requires gtest).
- **Python Tests**: `pytest -v python/cuopt/cuopt/tests` (core bindings).
- **Server Tests**: `pytest -v python/cuopt_server/tests` (REST API).

### 3. Quality Gate (Pre-commit)
Set up hooks once per clone: `pre-commit install`.
Hooks run automatically on `git commit`. If they fail, fix the issues before attempting to commit again.

## ⚡ Technical Deep-Dive

### 1. Python Bindings (Cython)
cuOpt bridges C++ and Python using Cython. 
- **Architecture**: Refer to [`references/python_bindings.md`](references/python_bindings.md) for the parameter flow between the Python API and the C library.
- **Pattern**: Follow existing Cython patterns when adding new API endpoints to ensure memory safety and performance.

### 2. VRP Dimension Implementation
When adding or modifying constraints, objectives, or propagation logic for Vehicle Routing Problems (VRP):
- **Required Reading**: [`references/vrp_skills.md`](references/vrp_skills.md) before implementation.
- **Checklist**: Ensure the new dimension implements the required interfaces and follows the `combine` semantics.

### 3. Numerical Debugging
For "wrong-but-plausible" outputs (e.g., invalid lower bounds, iteration blow-ups):
- **Methodology**: Use the workflow in [`resources/numerical_debugging.md`](resources/numerical_debugging.md).
- **Strategy**: Instrument the code first to find catastrophic cancellation sites $\rightarrow$ apply guards at the exact site. Avoid speculative patches.

## 🤝 Contribution & Governance

### 1. Commit & PR Rules
- **Sign-off**: Use `git commit -s` for DCO (Developer Certificate of Origin) compliance.
- **PR Strategy**: Use draft PRs for active development. Keep descriptions concise (avoid file tables).
- **Branching**: 
  - Development $\rightarrow$ `main`.
  - Current Release $\rightarrow$ `release/YY.MM` (check `git branch -r | grep release`).

### 2. Coding Conventions
Follow the standards in [`references/conventions.md`](references/conventions.md):
- **C++**: `snake_case`, `d_`/`h_` prefixes for device/host pointers, `_t` suffix for types.
- **Memory**: Follow RMM patterns; strictly no raw `new`/`delete`.
- **Error Handling**: Use `CUOPT_EXPECTS` and `RAFT_CUDA_TRY`.
