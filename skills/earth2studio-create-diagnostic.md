---
name: earth2studio-create-diagnostic
description: Rigorous pipeline for creating Earth2Studio diagnostic model wrappers (Simple, AutoModel, Generative) that perform single-step data transformations.
version: 0.16.0
category: data-science
---

# Create Diagnostic Model

The `earth2studio-create-diagnostic` skill defines the engineering standard for implementing diagnostic model wrappers in Earth2Studio. Unlike prognostic models, diagnostic models are restricted to single-step transformations (input $\rightarrow$ output) at a single point in time, ensuring a clean separation between state-less transformations and time-stepping forecasts.

## Invariants

- **Single-Step Only**: Diagnostic models must NEVER implement time-stepping or forecast integration. Any requirement for `lead_time` or iterative state updates must be routed to `earth2studio-create-prognostic`.
- **Strict Async/Batching**: All wrappers must use `@batch_func()` and `@batch_coords()` to ensure compatibility with Earth2Studio's high-throughput data pipeline.
- **Dependency Isolation**: AutoModel and Generative diagnostics MUST have a named optional dependency extra in `pyproject.toml` to prevent bloat in the core installation.
- **Coord-System Authority**: Input coordinates must strictly follow the Earth2Studio public order (Batch $\rightarrow$ Variable $\rightarrow$ Lat $\rightarrow$ Lon). `batch` must always be first and initialized as `np.empty(0)`.
- **Reference-Driven Implementation**: No model shall be implemented without a reference inference script, paper, or documentation to ensure tensor shape and normalization parity.

## Execution Pipeline

### 1. Classification & Dependency Planning
**Goal**: Determine the architecture and required environment extensions.
- **Type Mapping**:
    - **Simple**: Derived quantities, no checkpoints $\rightarrow$ `torch.nn.Module`.
    - **AutoModel**: Weights from Package/Checkpoint $\rightarrow$ `torch.nn.Module + AutoModelMixin`.
    - **Generative**: Diffusion/Stochastic outputs $\rightarrow$ `torch.nn.Module + AutoModelMixin` (adds `sample` dimension).
- **Extra Definition**: Propose a named extra (e.g., `model-precip = ["package>=1.0"]`) and update the `all` aggregate in `pyproject.toml`.

### 2. Implementation (Canonical Structure)
**Goal**: Build the wrapper using the repo-standard method ordering to ensure maintainability.
- **Method Order**: `__init__` $\rightarrow$ `input_coords` $\rightarrow$ `output_coords` $\rightarrow$ `load_default_package` $\rightarrow$ `load_model` $\rightarrow$ `to` $\rightarrow$ `__call__`.
- **Forward Pass**: Implement `__call__` decorated with `@torch.inference_mode()` and `@batch_func()`. It must return a tuple of `(output_tensor, output_coords)`.
- **Coord Validation**: Use `handshake_dim` and `handshake_coords` within `output_coords` to verify input tensor alignment before transformation.
- **Loading Logic**: Lock HuggingFace/S3 URLs to specific commits or versions. Load checkpoints on CPU first, set to `.eval()`, and disable gradients.

### 3. Testing & Quality Assurance
**Goal**: Prove the wrapper is robust and matches the reference implementation.
- **Test Matrix**:
    - `test_<model>_call`: Forward pass with a mock core model.
    - `test_<model>_exceptions`: Verify errors are raised for invalid coordinates/variables.
    - `test_<model>_package`: Real weight verification (using `@pytest.mark.package`).
- **Generative Specifics**: For generative models, implement deterministic-seed tests to ensure reproducibility.
- **Style Compliance**: Run `make format && make lint && make license`.

### 4. Integration & Registration
**Goal**: Export the model to the public API.
- **Registry**: Update `earth2studio/models/dx/__init__.py` alphabetically.
- **Documentation**: Add to `docs/modules/models_dx.rst` and update `docs/userguide/about/install.md` with `pip install` instructions for the model extra.
- **Change Log**: Record the addition in `CHANGELOG.md`.

## Gotchas

- **Coordinate Order Drift**: A common failure is placing `variable` before `batch` or mixing up North-to-South latitude conventions. Always verify against the `CoordSystem` standard.
- **Blocking I/O in `load_model`**: Loading huge checkpoints synchronously can hang the orchestrator. Use `loguru.logger` to signal loading progress.
- **Symptom: `OptionalDependencyFailure`**: Occurs when a model is called without its associated extra being installed. Fix via `uv sync --extra <model-name>`.
- **Random Input Failure**: Real checkpoints often fail on purely random tensors. Create "physically plausible" stable inputs for package tests.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `OptionalDependencyFailure` | Missing model extra | Check `pyproject.toml` $\rightarrow$ `uv pip list` | `uv add --extra <model-name>` |
| Coord handshake fails | Wrong `OrderedDict` order | Inspect `input_coords` $\rightarrow$ check `handshake_dim` | Reorder `input_coords` to match E2S standard |
| Wrong output tensor shape | `output_coords` mismatch | Compare returned tensor shape vs `output_coords` lengths | Fix `output_coords` logic |
| `ModuleNotFoundError: pytest` | Using system Python | Check `which pytest` | Use `uv run pytest` |
| Package test fails | Invalid input tensors | Check input range/normalization | Build stable, physically plausible input |
| `AttributeError` in `__call__` | Missing `@batch_func` | Check method decorators | Add `@batch_func()` and `@torch.inference_mode()` |

## Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `uv run pytest test/models/dx/test_<name>.py -m "not package" -v` | Fast mock tests | All tests passed (excluding package tests) |
| `uv run pytest ... --package -v` | Real weight tests | Successful forward pass with real checkpoint |
| `make format && make lint && make license` | Style/License check | `All checks passed` |
| `uv sync --all-extras` | Full environment sync | All optional dependencies installed |
