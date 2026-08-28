---
name: earth2studio-data-fetch
description: High-precision pipeline for identifying, verifying, and fetching weather/climate data via Earth2Studio data sources.
version: 0.16.0
category: data-science
---

# Earth2Studio Data Fetch

The `earth2studio-data-fetch` skill enables the precise retrieval of weather and climate data by mapping user requirements to the correct Earth2Studio data sources. It ensures that variable requests are physically supported by the source's lexicon before generating an executable fetch script.

## Invariants

- **Lexicon as Single Source of Truth (SSoT)**: No data source shall be recommended without verifying the requested variable against that source's specific lexicon file (e.g., `earth2studio/lexicon/<source>.py`). If a variable is not in the `VOCAB` dictionary, the source is incompatible.
- **Variable Mapping Standard**: Plain language requests (e.g., "geopotential height at 500hPa") MUST be mapped to Earth2Studio canonical names (e.g., `z500`) using the `E2STUDIO_VOCAB` in `lexicon/base.py` before processing.
- **Structural Separation**: Analysis sources (historical state) and Forecast sources (lead-time based) are strictly separated. A single fetch script must only use one source type.
- **Live Doc Verification**: Because API signatures evolve, the current documentation pages (`datasources_analysis.html`, `datasources_forecast.html`) must be consulted to confirm constructor arguments before script generation.

## Execution Pipeline

### 1. Requirement Extraction & Mapping
**Goal**: Translate a vague user request into a technical specification.
- **Parameter Extraction**: Identify Variables $\rightarrow$ Time (Range/Discrete) $\rightarrow$ Data Type (Analysis vs. Forecast) $\rightarrow$ Region $\rightarrow$ Output Format.
- **Canonical Mapping**: Convert requested variables to E2Studio canonical names using `lexicon/base.py`.

### 2. Candidate Identification & Lexicon Verification
**Goal**: Find the most efficient and compatible data source.
- **Candidate Filtering**: 
    - **Analysis**: GFS, HRRR, IFS, ARCO/CDS/WB2 (ERA5), GOES.
    - **Forecast**: GFS_FX, GEFS_FX, HRRR_FX, IFS_FX, AIFS_FX.
- **Lexicon Check**: For each candidate, check the specific lexicon file on GitHub.
    - **Logic**: If `var_name` $\in$ `Source.VOCAB` $\rightarrow$ Compatible.
- **Trade-off Analysis**: Compare candidates based on:
    - **Coverage**: Historical depth (e.g., ERA5 via ARCO goes back to 1959).
    - **Resolution**: Spatial granularity (e.g., HRRR 3km vs GFS 0.25°).
    - **Latency**: Update frequency and API throttling (e.g., CDS API queue vs GCS speed).

### 3. Source Selection & User Confirmation
**Goal**: Reach a consensus on the best data source.
- **Comparison Table**: Present viable options with their trade-offs (Variables, Coverage, Resolution, Time Range).
- **Recommendation**: Suggest the most optimal source (e.g., "Recommend ARCO for ERA5 as it is free and requires no API key").

### 4. Script Generation
**Goal**: Produce a production-ready Python script for data retrieval.
- **Pattern Selection**:
    - **Analysis**: `ds = SourceClass(); data = ds(time, variable)`
    - **Forecast**: `ds = SourceClass(); data = ds(time, lead_time, variable)`
- **Implementation Details**:
    - Include all necessary `datetime` and `earth2studio.data` imports.
    - Add comments explaining the `xarray.DataArray` output.
    - Include data inspection commands (`print(data.shape)`, `print(data.coords)`).
    - Implement optional saving to NetCDF/Zarr if requested.

### 5. Post-Fetch Guidance
**Goal**: Ensure the user can utilize the fetched data.
- **Cache Education**: Explain that data is cached locally via `EARTH2STUDIO_CACHE` to avoid redundant network calls.
- **Next Steps**: Point the user to the `earth2studio-discover` skill if they intend to feed this data into a model.

## Gotchas

- **The "CDS Queue" Trap**: The official Copernicus CDS API is often throttled by heavy queues. For ERA5 data, steer users toward ARCO (Google Cloud) or WB2 for significantly faster access.
- **Temporal Gaps**: Just because a source *supports* a variable doesn't mean the data exists for the requested time. Always warn the user that a `404` or `FileNotFoundError` may occur if the time is outside the source's operational window.
- **API Key Dependencies**: CDS-based sources require a `~/.cdsapirc` file. Ensure the user is aware of this prerequisite before recommending CDS.
- **Xarray Complexity**: The output is a high-dimensional `xarray.DataArray`. Users unfamiliar with `xarray` may struggle with slicing; provide a brief example of how to select a specific time/lat/lon.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `KeyError: '<var>'` | Variable not in Lexicon | Check `earth2studio/lexicon/<source>.py` | Try a different data source that supports the variable |
| `FileNotFoundError` / `404` | Time out of range | Check source's temporal coverage in docs | Adjust requested time range |
| `CDS API timeout` | Queue congestion | Check CDS API status | Switch to ARCO or WB2 for ERA5 data |
| `ModuleNotFoundError` | Earth2Studio not installed | `which python` $\rightarrow$ `pip list` | `uv pip install earth2studio` |
| Empty `DataArray` | Time/Variable mismatch | Inspect `data.coords` and `data.dims` | Verify datetime and variable name match source expectations |

## Command Appendix

| Tool/Resource | Purpose | Usage/Link |
| :--- | :--- | :--- |
| `base.py` | Canonical Variable Vocab | [GitHub Link](https://github.com/NVIDIA/earth2studio/blob/main/earth2studio/lexicon/base.py) |
| Source Lexicons | Source-specific support | `https://github.com/NVIDIA/earth2studio/blob/main/earth2studio/lexicon/<source>.py` |
| Analysis Docs | API for Analysis sources | [Web Link](https://nvidia.github.io/earth2studio/modules/datasources_analysis.html) |
| Forecast Docs | API for Forecast sources | [Web Link](https://nvidia.github.io/earth2studio/modules/datasources_forecast.html) |
