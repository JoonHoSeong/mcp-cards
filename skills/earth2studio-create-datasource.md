---
name: earth2studio-create-datasource
description: End-to-end workflow for implementing and validating new Earth2Studio data source wrappers (DataSource, ForecastSource, etc.) to connect remote stores to async fetching infra.
version: 0.16.0
category: data-science
---

# Create and Validate Data Source

The `earth2studio-create-datasource` skill provides a rigorous, 14-step engineering pipeline for integrating new remote data stores (S3, GCS, Azure, HTTP, HuggingFace) into the Earth2Studio ecosystem. It ensures that every new source is not only functional but also adheres to strict asynchronous I/O patterns, lexicon standards, and validation benchmarks.

## Invariants

- **Async-First I/O**: All data fetching must be implemented using pure async I/O. The use of `asyncio.to_thread` or bare `tqdm.gather` is strictly prohibited to prevent event loop blocking.
- **fsspec Preference**: `fsspec` is the authoritative filesystem abstraction. Dedicated libraries (e.g., `boto3`, `google-cloud-storage`) must only be used as a last resort when `fsspec` cannot access the store.
- **Lexicon-Driven Mapping**: Remote variable keys must never be hardcoded in the source. They must be mapped via a dedicated Lexicon class using the `E2STUDIO_VOCAB` standard.
- **Deliverables Co-Equality**: The test file (`test/data/test_<source>.py`) is a co-equal deliverable. A source is not considered "implemented" until its corresponding tests pass with >90% coverage.
- **No-Secret Commit**: Under no circumstances should credentials, API keys, or sanity-check scripts/images be committed to the repository.

## Execution Pipeline

### 1. Analysis & Type Determination
**Goal**: Define the mathematical and structural nature of the source.
- **Type Selection**: 
    - Gridded Analysis $\rightarrow$ `DataSource` (`xr.DataArray`)
    - Gridded Forecast $\rightarrow$ `ForecastSource` (`xr.DataArray` + `lead_time`)
    - Sparse/Station Obs $\rightarrow$ `DataFrameSource` (`pd.DataFrame`)
    - Sparse Forecast Obs $\rightarrow$ `ForecastFrameSource` (`pd.DataFrame` + `lead_time`)
- **Dependency Audit**: Analyze the remote store's protocol and propose the lightest `fsspec` backend (e.g., `s3fs`, `gcsfs`, `adlfs`).

### 2. Lexicon & Schema Definition
**Goal**: Map remote data to the Earth2Studio internal language.
- **Lexicon Implementation**: Create `earth2studio/lexicon/<source_name>.py` using `metaclass=LexiconType`.
- **Variable Mapping**: Map remote keys to `E2STUDIO_VOCAB` entries. Use `::` separators for structured/multi-dimensional keys.
- **Schema Update**: If the source is a `DataFrameSource`, update `E2STUDIO_SCHEMA` to include required fields.

### 3. Source Implementation (The Skeleton)
**Goal**: Build the async fetching engine following the canonical method order.
- **Method Ordering**: `__init__` $\rightarrow$ `_async_init` $\rightarrow$ `__call__` $\rightarrow$ `fetch` $\rightarrow$ `_create_tasks` $\rightarrow$ `fetch_wrapper` $\rightarrow$ `fetch_array` $\rightarrow$ `_validate_time`.
- **Async Patterns**: Implement `managed_session`, `gather_with_concurrency`, and `async_retry` to ensure resilience and performance.
- **Constructor Defaults**: Standardize on `cache=True`, `verbose=True`, `async_timeout=600`, `async_workers=16`, and `retries=3`.

### 4. Validation & Testing
**Goal**: Prove the source is robust and the data is physically reasonable.
- **Test Suite**: Implement `test_<source>_fetch` (slow), `_cache` (slow), `_call_mock`, and `_exceptions`.
- **Style Compliance**: Execute `make format && make lint && make license`.
- **Sanity-Check Plots**: Generate visual plots of the fetched data. **The user must visually inspect and confirm these plots before PR submission.**
- **Variable Audit**: Remove any variables with $<10\%$ valid data across the requested range.

### 5. Integration & Submission
**Goal**: Formalize the addition to the codebase.
- **Registration**: Add alphabetical imports to `earth2studio/data/__init__.py` and `earth2studio/lexicon/__init__.py`.
- **Documentation**: Update relevant `.rst` files and provide NumPy-style docstrings with reference URLs and download size warnings.
- **PR Process**: Create `feat/data-source-<name>` branch $\rightarrow$ Open PR $\rightarrow$ Post validation summary as a comment.
- **Automated Review**: Triage Greptile feedback (Bug $\rightarrow$ Style $\rightarrow$ Perf $\rightarrow$ Docs $\rightarrow$ Suggestion).

## Gotchas

- **System Python Contamination**: Never use system Python. Always use `uv run python` or the local `.venv` to avoid dependency drift.
- **Xarray Loading Bottleneck**: Avoid using `xarray` for the initial loading of remote data; use pure async I/O and only wrap the final result in an `xr.DataArray`.
- **Paging/Chunking Failures**: For very large remote stores, ensure `_create_tasks` correctly chunks requests to avoid hitting remote API rate limits.
- **Time-Zone Drift**: When implementing `_validate_time`, ensure the remote store's temporal resolution matches the requested Earth2Studio timestamps to avoid off-by-one errors.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Fix / Next Step |
| :--- | :--- | :--- | :--- |
| `ImportError` for backend | Missing `fsspec` extension | Check `pyproject.toml` $\rightarrow$ `uv pip list` | `uv add --extra data <package>` |
| `asyncio.TimeoutError` | Remote store latency / too many workers | Reduce `async_workers` $\rightarrow$ increase `async_timeout` | Adjust constructor params |
| Lexicon `KeyError` | Mapping mismatch with remote keys | Run lexicon debug script $\rightarrow$ check remote API | Update `VOCAB` in lexicon class |
| `pytest` fails on `_call_mock` | Async wrapper logic error | Inspect `fetch_wrapper` $\rightarrow$ check `_sync_async` | Fix `managed_session` implementation |
| Sanity plot is empty/NaN | Incorrect variable mapping or empty store | Check `_validate_time` $\rightarrow$ verify raw API response | Update variable keys in Lexicon |
| Greptile flags "Blocking I/O" | Use of `requests` or `boto3` in async path | Search for synchronous calls in `fetch` | Replace with `httpx` or `fsspec` async |

## Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `uv run python` | Run Python via uv | Standard Python REPL/Script output |
| `uv add --extra data <package>` | Add optional data dependency | `pyproject.toml` and `uv.lock` updated |
| `make format && make lint` | Style check | `All checks passed` |
| `uv run pytest test/data/test_<source>.py -x` | Run unit tests | `100% passed` |
