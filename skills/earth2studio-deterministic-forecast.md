---
name: earth2studio-deterministic-forecast
description: Comprehensive pipeline for building single-member deterministic weather forecast scripts using the `earth2studio.run.deterministic` orchestrator.
version: 0.16.0
category: data-science
---

# Earth2Studio Deterministic Forecast

The `earth2studio-deterministic-forecast` skill enables the creation of a complete inference pipeline that combines a prognostic model, a data source, and an IO backend to generate a deterministic weather forecast. It focuses on the orchestration of the "Initial Condition $\rightarrow$ Model $\rightarrow$ Storage" flow.

## Invariants

- **Prognostic-Data Compatibility**: A data source MUST be verified against the model's `input_coords["variable"]` via the lexicon (`earth2studio/lexicon/<source>.py`). Any mismatch results in a runtime `KeyError` during the first step.
- **Time-Step Synchronization**: The number of steps (`nsteps`) must be an exact integer division: `nsteps = forecast_hours // model_step_hours`. Fractional steps are not supported.
- **CUDA Requirement**: All prognostic models in Earth2Studio require a CUDA-capable GPU. CPU-only inference is an unsupported state.
- **ISO 8601 Authority**: The `time` argument passed to the orchestrator MUST follow the ISO 8601 format (`YYYY-MM-DDTHH:MM:SS`).
- **Weight Loading Standard**: Models MUST be loaded using the `ModelClass.load_model(ModelClass.load_default_package())` pattern to ensure weights are pulled from the immutable default registry.

## Execution Pipeline

### 1. Requirement Specification
**Goal**: Define the forecast's temporal and spatial boundaries.
- **Parameter Extraction**: Forecast Horizon (Total Hours) $\rightarrow$ Variables of Interest $\rightarrow$ Target Region $\rightarrow$ Available VRAM.
- **Variable Expansion**: If the user requests "wind speed", automatically expand this to both `u10m` and `v10m` components.

### 2. Component Selection & Verification
**Goal**: Assemble a compatible set of model, data, and storage components.
- **Model Selection**: Filter prognostic models by horizon, region, and VRAM. Extract the model's `time_step` (e.g., 6h).
- **Data Source Verification**: 
    - Check the model's required input variables.
    - Verify these variables exist in the candidate data source's `VOCAB` lexicon.
    - Common pairs: Global $\rightarrow$ GFS/ARCO; Regional $\rightarrow$ HRRR.
- **IO Backend Selection**:
    - `ZarrBackend` (Default): Optimized for cloud/large-scale storage.
    - `NetCDF4Backend`: For legacy tool compatibility.
    - `XarrayBackend`: For small-scale, in-memory experimentation.

### 3. Step Calculation & Output Filtering
**Goal**: Configure the temporal depth and data volume of the output.
- **Step Logic**: Calculate `nsteps = total_hours // model_step_hours`.
- **Output Optimization**:
    - If specific variables are requested $\rightarrow$ Define `output_coords = OrderedDict({"variable": np.array([vars])})`.
    - If "all variables" are requested $\rightarrow$ Omit `output_coords` to preserve full model output.

### 4. Script Generation
**Goal**: Produce a production-ready inference script.
- **Orchestrator Pattern**: Use `earth2studio.run.deterministic`.
- **Structural Requirements**:
    - Load model via `load_default_package()`.
    - Initialize `DataSourceClass` and `IOBackendClass`.
    - Pass `time` (ISO 8601), `nsteps`, `prognostic` (model), `data` (source), `io` (backend), and `device` (cuda).

### 5. Manual Loop Implementation (Optional)
**Goal**: Provide an alternative for users requiring fine-grained control.
- **Strict Sequence**:
    1. `fetch_data`: Retrieve initial conditions.
    2. `total_coords`: Pre-calculate the full coordinate grid for the forecast horizon.
    3. `io.add_array`: Initialize the backend storage with the total grid.
    4. `create_iterator`: Initialize the prognostic model's stateful iterator.
    5. **Loop**: Iterate through `nsteps`, calling `map_coords` $\rightarrow$ `split_coords` $\rightarrow$ `io.write` at each step.

## Gotchas

- **The "Missing Variable" Crash**: Recommending a model and a data source that seem related but have slightly different variable names in their lexicons. Always verify the *exact* strings in the `.py` lexicon files.
- **VRAM OOM**: Large models (e.g., high-res global models) can easily exceed 24GB VRAM during the first step. Always check the model's VRAM requirements against the target hardware.
- **Zarr Lock/Corruption**: Writing to a Zarr store that is currently being read by another process can lead to corruption. Ensure the `output_path` is unique or the store is closed.
- **Symptom: "Empty Output"**: Often caused by an incorrect `nsteps` calculation or an incorrectly specified `time` that falls outside the data source's available window.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `KeyError: '<var>'` | Lexicon mismatch | Compare model `input_coords` vs source `VOCAB` | Change data source or map variables |
| `OutOfMemoryError` | VRAM exhausted | Check `nvidia-smi` during the first step | Reduce batch size or use a smaller model |
| `ValueError` in `deterministic()` | Invalid `time` format | Check if `time` is ISO 8601 string | Convert `datetime` to `YYYY-MM-DDTHH:MM:SS` |
| `FileNotFoundError` | Data not available | Check source temporal coverage in docs | Adjust forecast initialization time |
| `RuntimeError` (CUDA) | Device mismatch | Check `device` argument in `deterministic()` | Ensure `torch.device("cuda")` is used |

## Command Appendix

| Component | Recommendation | Documentation Link |
| :--- | :--- | :--- |
| Model | Based on Horizon/VRAM | [Models PX Docs](https://nvidia.github.io/earth2studio/modules/models_px.html) |
| Data Source | Based on Lexicon | [Analysis Sources](https://nvidia.github.io/earth2studio/modules/datasources_analysis.html) |
| IO Backend | `ZarrBackend` (Default) | [IO Modules Docs](https://nvidia.github.io/earth2studio/modules/io.html) |
| Orchestrator | `run.deterministic` | [run.py Source](https://github.com/NVIDIA/earth2studio/blob/main/earth2studio/run.py) |
