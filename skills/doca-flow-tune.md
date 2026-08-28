---
name: doca-flow-tune
category: devops
description: Analyze and optimize DOCA Flow pipeline performance by identifying bottlenecks and applying tuning recommendations using the `doca_flow_tune` tool.
trigger: "User wants to optimize a live DOCA Flow application, identify hardware/software bottlenecks in a Flow pipeline, or apply tuning recommendations to improve throughput/latency."
---

# DOCA Flow Tune (doca-flow-tune) Skill Card

## Overview
`doca_flow_tune` is an optimization tool that analyzes DOCA Flow pipelines to identify bottlenecks and provide specific tuning recommendations. It bridges the gap between raw performance measurement (`doca-flow-perf`) and live application optimization.

### Core Value Proposition
- **Bottleneck Identification**: pinpoint whether a pipeline is limited by hardware resources or software configuration.
- **Recommendation Engine**: Provides actionable tuning suggestions based on analyzed pipeline state.
- **Multi-Mode Analysis**: Supports offline analysis of captured configurations and online analysis of live pipelines.
- **Safe Optimization**: Implements a strict "smoke-before-bulk" gate for applying state-changing recommendations.

## Implementation Path: The Tuning Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Environment Match**: Verify the four-way match (Command line + JSON config + DOCA version + Device + Env).
2. **Tool Discovery**: Confirm `doca_flow_tune` binary and the shipped `flow_tune_cfg*.json` templates are present.
3. **Version Alignment**: Ensure the tune binary's version matches the `doca-flow` library version used by the application.
4. **Template Selection**: Use the shipped JSON templates as the schema source of truth for creating tuning configurations.
5. **Baseline Integration**: Load `doca-flow-perf` or `doca-flow-dpa-perf` baselines to provide the "before" context for the optimization.

### 2. Execution & Analysis (`## run`)
1. **Offline Analysis**:
   - Run `doca_flow_tune` against a captured pipeline-description JSON to generate analyze/visualize outputs without impacting live traffic.
2. **Online Read-Only Snapshot**:
   - Snapshot a live pipeline using a JSON config naming the server UDS and scope.
3. **Post-Processing**:
   - Use `scripts/hw_counters_csv_analyzer.py` to analyze the dumper CSV.
   - Use `scripts/flow_json_diff.py` and `scripts/flow_mermaid_diff.py` to compare sessions.

### 3. Optimization & Application (`## modify`)
1. **Recommendation Review**: Analyze the recommendation in the context of the chosen three-axis decision.
2. **Actionability Check**: Confirm the recommendation maps to a knob the Flow program actually exposes.
3. **Safe Application**:
   - **Rule**: Every state-changing operation must be gated by a clean smoke test.
   - **Action**: Apply the recommendation via a "modify-a-sample" Flow-program change.

## Critical Rules & Safety

### 1. The "Smoke-before-Bulk" Mandate
**No state-changing recommendation may be applied without a preceding clean smoke test.**
- **Rule**: Applying tuning changes directly to a production pipeline can cause dataplane disruption. The smoke test is the non-negotiable gate.

### 2. The "Four-Tuple" Context Requirement
**A tuning recommendation without its full context is meaningless.**
- **Rule**: Every recommendation must be quoted alongside its Four-Tuple: (1) Command line, (2) JSON config, (3) DOCA version, (4) Device + Env.

### 3. "Quote, Do Not Paraphrase"
**Tuning data requires absolute fidelity.**
- **Rule**: Quote the dumper CSV summary and analyze JSON verbatim. Do not summarize recommendations into prose or reorder Mermaid diagrams.

### 4. Role Explicitly
**Clearly distinguish between the tool's internal roles.**
- **Rule**: The agent must explicitly state if a session is "Offline", "Online Read-Only", or "Online State-Changing" before invocation.

## Diagnostic Ladder (Overlay)

| Layer | Tune Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Install)** | Binary or templates not found. | Route to `doca-setup`. |
| **Layer 2 (Policy)** | JSON config parse error. | Compare against shipped `flow_tune_cfg*.json` templates. |
| **Layer 3 (Analysis)** | Measurement is unsound (warm-up/steady-state issues). | Verify warm-up period and capture a before/after pair. |
| **Layer 4 (Action)** | Recommendation is unactionable (no program knob). | Propose a Flow-program change instead of a tune re-run. |
| **Layer 5 (Version)** | Tune binary version mismatch with library. | Walk the `doca-version` debug ladder. |
| **Layer 6 (Cross)** | Env-side errors (hugepages, drivers). | Route to `doca-debug` and `doca-setup`. |

## Command Appendix

### Flow Tune Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Purpose | Command (Class Shape) | Owning Step | Healthy Output |
| --- | --- | --- | --- |
| CLI Discovery | `doca_flow_tune --help` | `## configure` | Documented mode/scope flags. |
| Template Access | Inspect `flow_tune_cfg*.json` templates | `## configure` | Valid schema for tuning JSON. |
| Library Version | `pkg-config --modversion doca-flow` | `## configure` | Matches tune binary version. |
| Offline Analysis | `doca_flow_tune` (Offline mode) | `## run` | Analyze JSON and Visualize Mermaid. |
| Online Snapshot | `doca_flow_tune` (Online Read-Only) | `## run` | Dumper CSV and Visualize Mermaid. |
| Session Diff | `scripts/flow_json_diff.py` / `scripts/flow_mermaid_diff.py` | `## test` | Structural and counter diffs. |
| CSV Analysis | `scripts/hw_counters_csv_analyzer.py` | `## run` | Per-counter statistics. |
| Session Snapshot | Capture `outputs_directory` + Four-Tuple | `## test` | Verbatim bundle for downstream debug. |

## Deferred Topic Boundaries
- **Baseline Measurement**: Measuring primary performance numbers is handled by `doca-flow-perf` or `doca-flow-dpa-perf`.
- **Pipeline Creation**: Writing the original Flow application is handled by `doca-flow`.
- **Environment Setup**: Firmware/driver updates are handled by `doca-setup` and `doca-version`.
