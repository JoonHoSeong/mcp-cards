---
name: earth2studio-discover
description: High-precision discovery and compatibility engine for Earth2Studio models, data sources, and examples.
version: 0.16.0
category: data-science
---

# Earth2Studio Discover

The `earth2studio-discover` skill is a specialized discovery engine designed to map user weather/climate use cases to the most compatible Earth2Studio components. Instead of relying on static lists, it uses a live-documentation-first approach to ensure recommendations are always current and technically verified.

## Invariants

- **Live-Doc Primacy**: No component shall be recommended based on internalized memory. All recommendations MUST be preceded by a fetch of the current live documentation pages (e.g., `models_px.html`, `datasources_analysis.html`).
- **Badge-Based Filtering**: Recommendations MUST be filtered by the four canonical Earth2Studio badges: **Class** (e.g., MR, S2S), **Region** (e.g., Global, NA), **VRAM** (e.g., 40GB, 80GB), and **Release Year**.
- **Lexicon-Verified Compatibility**: Compatibility between a model and a data source is NOT assumed. It MUST be verified by confirming that the model's `input_coords["variable"]` list is a subset of the data source's `VOCAB` keys in the specific lexicon file (`earth2studio/lexicon/<source>.py`).
- **Triage-Driven Discovery**: Discovery must follow a structured path: Task Type $\rightarrow$ Hardware Constraints $\rightarrow$ Regional Needs $\rightarrow$ Variable Compatibility.

## Execution Pipeline

### 1. Use Case Decomposition
**Goal**: Translate a vague request into a technical discovery profile.
- **Dimension Extraction**:
    - **Task Type**: Nowcasting, Medium-Range (MR), Seasonal (S2S), Downscaling, Data Assimilation (DA), or Climate Projection.
    - **Region**: Global vs. Regional (e.g., CONUS, Europe).
    - **Temporal Scale**: Hours (Nowcast), Days (MR), Weeks/Months (S2S).
    - **Hardware**: Available GPU VRAM (e.g., 48GB, 80GB).
    - **Mode**: Deterministic vs. Ensemble.

### 2. Live Component Discovery
**Goal**: Identify candidates via current documentation.
- **Model Discovery**:
    - Fetch appropriate page: `models_px.html` (Prognostic), `models_dx.html` (Diagnostic), or `models_da.html` (Assimilation).
    - Extract candidate models matching the user's **Region** and **VRAM** constraints.
- **Data Source Discovery**:
    - Fetch `datasources_analysis.html` (Historical) or `datasources_forecast.html` (Operational).
    - Identify sources covering the target region and requested variables.

### 3. Technical Compatibility Verification
**Goal**: Prove the model-data pairing is executable.
- **The Lexicon Check**:
    1. Extract the `input_coords` variable list from the model's documentation or source.
    2. Fetch the corresponding lexicon file: `earth2studio/lexicon/<source>.py`.
    3. **Verification Logic**: `set(model_vars) ⊆ set(source_vocab_keys)`.
    4. **Result**: If the condition is false, the data source is incompatible, regardless of regional overlap.

### 4. Example Mapping
**Goal**: Provide a concrete implementation starting point.
- **Pattern Matching**: Match the user's task to the Examples Gallery categories:
    - `01_getting_started` $\rightarrow$ Basic pipelines.
    - `02_medium_range` $\rightarrow$ Ensemble/Perturbation.
    - `03_downscaling` $\rightarrow$ Super-resolution (CorrDiff, etc.).
    - `04_nowcasting` $\rightarrow$ StormCast/StormScope.
- **Selection**: Recommend 1-3 specific examples that mirror the intended workflow.

### 5. Recommendation Delivery
**Goal**: Present a concise, verified selection.
- **Output Format**:
    - **Use Case Summary**: Restate the technical profile.
    - **Model Short-list**: Table including [Model | Class | Region | VRAM | Rationale].
    - **Verified Data Sources**: Table including [Source | Coverage | Compatible Model].
    - **Curated Examples**: [Example Name $\rightarrow$ Link $\rightarrow$ Purpose].

## Gotchas

- **The "Static Memory" Trap**: Earth2Studio's catalog changes every release. Recommending a model that was removed or renamed in the latest version is a critical failure. Always fetch live docs.
- **VRAM Underestimation**: A model may fit in VRAM for a single step but OOM during an ensemble run or high-resolution forecast. Always add a 10-20% safety buffer to the Rec VRAM badge.
- **Lexicon-Doc Mismatch**: Occasionally, the documentation page lists a variable that hasn't been added to the `.py` lexicon file yet. The `.py` lexicon file is the absolute source of truth.
- **Symptom: "No Compatible Source"**: Often occurs when the user requests a high-resolution regional model (e.g., HRRR) but wants global data. Steer them toward a global model (e.g., GraphCast) or a compatible regional source.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| Model page 404 | Release-based URL change | Check main `nvidia.github.io/earth2studio/` nav | Update URL from the main index |
| Lexicon file missing | New/Renamed data source | Search `earth2studio/lexicon/` directory | Find the current filename via directory listing |
| Model VRAM OOM | Underestimated overhead | Check `nvidia-smi` during initialization | Recommend a smaller model or higher-VRAM GPU |
| Source incompatible | Variable missing in VOCAB | Check `earth2studio/lexicon/<source>.py` | Switch to a source that supports all required vars |
| Example broken | API drift since example creation | Check example's `requirements.txt` vs current version | Adapt example code to the latest API |

## Command Appendix

| Resource | Purpose | Live Link |
| :--- | :--- | :--- |
| Prognostic Models | Model discovery | [models_px.html](https://nvidia.github.io/earth2studio/modules/models_px.html) |
| Diagnostic Models | Model discovery | [models_dx.html](https://nvidia.github.io/earth2studio/modules/models_dx.html) |
| Analysis Sources | Data discovery | [datasources_analysis.html](https://nvidia.github.io/earth2studio/modules/datasources_analysis.html) |
| Forecast Sources | Data discovery | [datasources_forecast.html](https://nvidia.github.io/earth2studio/modules/datasources_forecast.html) |
| Lexicon Root | Technical verification | [GitHub Lexicon Dir](https://github.com/NVIDIA/earth2studio/tree/main/earth2studio/lexicon) |
