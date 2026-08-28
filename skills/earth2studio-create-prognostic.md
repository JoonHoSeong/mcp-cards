---
name: earth2studio-create-prognostic
description: Rigorous pipeline for implementing Earth2Studio prognostic (time-stepping) model wrappers, enabling forward-integrated weather forecasts.
version: 0.16.0
category: data-science
---

# Create Prognostic Model

The `earth2studio-create-prognostic` skill defines the engineering standard for implementing prognostic model wrappers in Earth2Studio. Unlike diagnostic models, prognostic models are designed for time-integration; they take an initial state and predict future states by stepping forward through time (e.g., 6-hour increments), requiring a stateful iteration loop.

## Invariants

- **Triple Inheritance**: Every prognostic model MUST inherit from `torch.nn.Module`, `AutoModelMixin`, and `PrognosticMixin`. This ensures compatibility with the base model logic, automatic weight management, and time-stepping infrastructure.
- **Time-Stepping Obligation**: A prognostic model is not complete without a functional `create_iterator` that yields the initial condition (Step 0) followed by subsequent predicted states.
- **Coordinate Convention Authority**: All models must expose latitude as North-to-South (90 to -90) and longitude as 0 to 360. Internal tensor flipping must be handled privately within the wrapper to maintain the public Earth2Studio convention.
- **Dependency Isolation**: Every prognostic model MUST have a named optional dependency extra in `pyproject.toml`, even if the list is empty, to support the `OptionalDependencyFailure` and `@check_optional_dependencies` patterns.
- **Reference-First Implementation**: Implementation must be driven by a reference inference script or paper to ensure exact parity in tensor shapes, normalization, and time-step increments.

## Execution Pipeline

### 1. Analysis & Dependency Planning
**Goal**: Define the model's temporal behavior and environment requirements.
- **Reference Audit**: Extract input/output variables, tensor shapes, time-step size (e.g., 6h), and checkpoint source.
- **Extra Definition**: Propose a named extra (e.g., `model-pangu = ["package>=1.0"]`) and update the `all` aggregate in `pyproject.toml`.
- **Approval**: Present the dependency list and time-step logic for user confirmation before editing files.

### 2. Implementation (Canonical Structure)
**Goal**: Build the wrapper following the repo-standard method ordering for maintainability.
- **Method Order**: `__init__` $\rightarrow$ `input_coords` $\rightarrow$ `output_coords` $\rightarrow$ `load_default_package` $\rightarrow$ `load_model` $\rightarrow$ `to` $\rightarrow$ `__call__` $\rightarrow$ `_default_generator` $\rightarrow$ `create_iterator`.
- **Coordinate Setup**: 
    - `input_coords`: Set `batch` as `np.empty(0)`, `time` as `np.empty(0)`, and `lead_time` starting at `np.timedelta64(0, "h")`.
    - `output_coords`: Use `handshake_dim` to validate inputs, then increment `lead_time` by the model's specific step.
- **Forward Pass**: Implement `__call__` decorated with `@batch_func`. It must handle the reshape from E2S format to model format and back.
- **Iteration Logic**: Implement `create_iterator` to yield the initial state, then loop using `front_hook` $\rightarrow$ `self()` $\rightarrow$ `rear_hook`.

### 3. Loading & Weight Management
**Goal**: Ensure immutable and efficient model loading.
- **Package Locking**: Use immutable URLs (e.g., `hf://org/repo@commit`) in `load_default_package`.
- **Resource Efficiency**: Load checkpoints on CPU first, set to `.eval()`, and disable gradients.
- **Dependency Guard**: Decorate `load_model` with `@check_optional_dependencies()` to trigger clear errors if the model extra is missing.

### 4. Validation & Testing
**Goal**: Prove the wrapper is robust and mathematically correct.
- **Test Matrix**:
    - `test_<model>_call`: Verify a single forward pass across devices.
    - `test_<model>_iter`: Ensure the iterator produces the correct sequence of states.
    - `test_<model>_exceptions`: Verify errors for invalid coordinates or variables.
    - `test_<model>_package`: Real weight verification using `@pytest.mark.package`.
- **Style Compliance**: Execute `make format && make lint && make license`.
- **Sanity-Check Plots**: Generate visual comparisons between the wrapper and the reference implementation. **User visual confirmation is mandatory before PR submission.**

### 5. Integration & Registration
**Goal**: Formalize the model's availability in the public API.
- **Registry**: Update `earth2studio/models/px/__init__.py` alphabetically.
- **Documentation**: Add to `docs/modules/models_px.rst` and provide installation instructions in `docs/userguide/about/install.md`.
- **Change Log**: Record the addition in `CHANGELOG.md`.

## Gotchas

- **Coordinate Order Drift**: A common failure is placing `time` or `lead_time` before `batch`. Always verify against the `CoordSystem` standard.
- **The "Step 0" Omission**: Forgetting to yield the initial condition in `create_iterator` breaks all downstream forecasting pipelines.
- **Latitude Flipping**: If the core model expects South-to-North, flipping the tensor only in `input_coords` is incorrect; it must be flipped internally before the call and flipped back before returning.
- **Symptom: `OptionalDependencyFailure`**: Occurs when the model extra was not installed. Fix via `uv sync --extra <model-name>`.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `OptionalDependencyFailure` | Missing model extra | Check `pyproject.toml` $\rightarrow$ `uv pip list` | `uv add --extra <model-name>` |
| Coord handshake fails | Wrong `OrderedDict` order | Inspect `input_coords` $\rightarrow$ check `handshake_dim` | Reorder `input_coords` to match E2S standard |
| Iterator produces wrong shapes | Reshape logic error in `__call__` | Print tensor shapes before/after model call | Fix reshape/permute logic in `__call__` |
| `ModuleNotFoundError: pytest` | Using system Python | Check `which pytest` | Use `uv run pytest` |
| Package test fails | Invalid input tensors | Check input range/normalization | Build stable, physically plausible input |
| `AttributeError` in `create_iterator` | Missing `PrognosticMixin` | Check class inheritance | Add `PrognosticMixin` to class definition |

## Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `uv run pytest test/models/px/test_<name>.py -v` | Full unit test suite | All tests passed |
| `uv run pytest ... --package -v` | Real weight verification | Successful forward pass with real checkpoint |
| `make format && make lint && make license` | Style/License check | `All checks passed` |
| `uv sync --all-extras` | Full environment sync | All optional dependencies installed |
