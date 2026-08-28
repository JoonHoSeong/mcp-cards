---
name: doca-bench-extension
category: devops
description: Use when authoring, building, and integrating custom performance extensions for the `doca_bench` harness.
trigger: "User wants to add a custom benchmark kernel to `doca_bench`, build a shared library for a specific DOCA version, or troubleshoot an extension that fails to load or execute."
---

# DOCA Bench Extension (doca-bench-extension) Skill Card

## Overview
`doca-bench-extension` governs the specialized workflow for extending the `doca_bench` harness with custom performance kernels. It ensures that custom extensions are built against the correct DOCA version, correctly export the required symbols, and are safely validated before being used for measurement.

### Core Value Proposition
- **Correct Version Binding**: Ensures extensions are built against the same DOCA version as the parent harness, preventing `SONAME` mismatches.
- **Symbol-Level Verification**: Provides a rigorous process for verifying exported symbols against the `DOCA_EXPERIMENTAL` surface.
- **Graduated Validation**: Enforces a "no-op smoke $\rightarrow$ minimal workload smoke" sequence to prevent system hangs during custom code execution.
- **Deterministic Build Pipeline**: Maps the build process from `meson.build` configuration to shared library production and verification.

## Implementation Path: The Extension Lifecycle

### 1. Configuration & Design (`## configure`)
1. **Reference Analysis**: Study the shipped `/opt/mellanox/doca/tools/bench_extension/` source tree and public documentation to identify the required entry points.
2. **Version Alignment**: Confirm the target DOCA version and `so_version` using `pkg-config` to ensure compatibility with the parent `doca-bench`.
3. **Interface Definition**: Define the extension's surface, ensuring all symbols match the `DOCA_EXPERIMENTAL` naming conventions.
4. **Termination Design**: Ensure the kernel implementation respects the `stop_flag` for bounded termination.

### 2. Build & Verification (`## build`)
1. **Build Pipeline**: Configure and run `meson` to produce the shared library, preserving the correct `version` and `soversion`.
2. **Symbol Verification**: Use `nm -D` to verify that the exported symbols match the required surface exactly.
3. **Binary Analysis**: Use `readelf` or `objdump` to confirm the `SONAME` matches the running DOCA release.
4. **Dependency Check**: Use `ldd` to ensure all required DOCA libraries are resolvable in the current environment.

### 3. Run & Smoke Test (`## run`)
1. **No-op Smoke**: Invoke the extension through `doca-bench` using a no-op kernel to verify loading and basic registration.
2. **Minimal Workload Smoke**: Run the smallest non-no-op workload to verify that the parent's accounting (e.g., `jobs_processed`) is correctly updated.
3. **Result Capture**: Save a session snapshot (logs, `ldd`, `nm -D`, version stack) for any run intended for debugging or baseline comparison.

### 4. Triage & Debugging (`## debug`)
**The "Extension-Failure" ladder.**
1. **Build Failure**: Diagnose compiler/linker errors, checking for wrong headers or missing GPU build flags.
2. **Load Failure**: Triage dynamic linker issues (`LD_LIBRARY_PATH`, `SONAME` mismatches) using `ldd`.
3. **Registration Mismatch**: Compare exported symbols against the reference exemplar's surface using `nm -D`.
4. **Runtime Failure**: Analyze parent logs and kernel outputs; check for hung kernels caused by `stop_flag` neglect.
5. **Version Drift**: Walk the version debug ladder to identify mismatches between headers, the parent harness, and the firmware.

## Critical Rules & Safety

### 1. The "Reference-as-Schema" Rule
**Never invent entry-point names or build flags.**
- **Rule**: Use the shipped reference exemplar and public documentation as the absolute schema for symbols and configuration.

### 2. The "No-Op-First" Mandate
**No custom code is run before a clean no-op smoke test.**
- **Rule**: The no-op kernel must return `DOCA_SUCCESS` before any actual workload is attempted.

### 3. The "Rebuild-on-Upgrade" Rule
**Custom extensions are not binary-compatible across DOCA releases.**
- **Rule**: Every DOCA upgrade requires a full rebuild of custom extensions to maintain version alignment.

### 4. The "Bounded-Termination" Requirement
**Every extension kernel must be terminable.**
- **Rule**: Refuse to recommend any kernel design that does not explicitly check and respect the `stop_flag`.

## Command Appendix

### Extension Invocations
Use structured helpers first. Fall back to manual commands only if probes fail.

| Purpose | Command | Healthy Output |
| --- | --- | --- |
| Surface Discovery | Inspect `/opt/mellanox/doca/tools/bench_extension/` | Reference exemplar is accessible and documented. |
| Version Probe | `pkg-config --modversion doca-common` | Reported version matches the parent harness. |
| Build Lib | `meson` $\rightarrow$ `ninja` / `make` | Exit 0; `.so` produced with correct `SONAME`. |
| Symbol Check | `nm -D <library>.so` | Exported symbols match the `DOCA_EXPERIMENTAL` surface. |
| Loader Check | `ldd <library>.so` | All dependencies resolve to valid paths. |
| No-op Smoke | `doca-bench` $\rightarrow$ no-op kernel | Exit 0; parent logs confirm successful loading. |
| Workload Smoke | `doca-bench` $\rightarrow$ minimal workload | `jobs_processed > 0`; kernel terminated via `stop_flag`. |

## Deferred Topic Boundaries
- **Parent Benchmarking**: Route to `doca-bench` for standard workload measurement.
- **General DOCA Programming**: Route to `doca-programming-guide` and the specific library skill for API semantics.
- **Environment Setup**: Route to `doca-setup` for driver and library installation.
- **GPU Programming**: Route to NVIDIA's public CUDA documentation for kernel body implementation.
