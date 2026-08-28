---
name: nemo-relay-plugin-adaptive-tuning
description: Configure and evaluate adaptive plugin behavior in NeMo Relay, including telemetry, state, adaptive hints, tool parallelism, and the Adaptive Cache Governor (ACG).
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Adaptive Plugin Tuning Guide

This skill provides the operational framework for optimizing NeMo Relay runtimes using the `adaptive` plugin component. Adaptive tuning uses runtime signals (telemetry) to dynamically adjust behavior such as tool parallelism, prompt-caching, and routing.

## 1. Adaptive Model Fundamentals
Adaptive behavior is implemented as a first-party plugin component (`kind: "adaptive"`). It learns from existing NeMo Relay scope events and managed tool/LLM lifecycle streams.

### Core Configuration Areas
- **State Management**: 
    - `in_memory`: Fast, volatile; used for local development.
    - `redis`: Persistent and shared; required for multi-worker environments.
- **Tool Parallelism**:
    - `observe_only`: Collects telemetry without acting.
    - `inject_hints`: Suggests parallelism to the app logic.
    - `schedule`: Actively manages the execution schedule.
- **Adaptive Cache Governor (ACG)**: Optimizes prompt-cache planning.
    - Providers: `passthrough`, `anthropic`, `openai`.
- **Rollout Policy**: Controls how learned behaviors are applied to traffic.

## 2. Operational Rollout Sequence
Adaptive tuning must be performed iteratively against a known baseline to avoid introducing instability.

1. **Signal Verification**: Confirm that the application is emitting the required scope and tool/LLM events.
2. **Baseline Capture**: Measure current latency, correctness, and failure rates.
3. **Telemetry-Only Phase**: Enable adaptive telemetry with `in_memory` state to observe runtime patterns without affecting execution.
4. **Iterative Activation**:
    - Enable the smallest possible behavior change (e.g., `inject_hints`).
    - Verify tool idempotency before enabling `schedule` mode.
    - Verify provider payload stability before enabling ACG.
5. **Comparative Analysis**: Compare results against the baseline. If regression occurs, revert to the last known working configuration.

## 3. Critical Pitfalls & Safety Rails
- **Scheduling Risks**: Never enable `schedule` mode before verifying that the target tools are idempotent and race-condition free.
- **Cache Planning**: Do not enable ACG if the provider request payloads are unstable, as this will lead to cache misses or incorrect routing.
- **Hint Consumption**: Treat adaptive hints as *suggestions*, not mandatory instructions, unless the consuming application logic explicitly defines a mandatory contract.
- **Configuration Anti-Patterns**: Do not use environment variables for primary adaptive tuning; use the structured plugin configuration document.
- **Data Sensitivity**: Do not tune based on a single run or unrepresentative traffic.

## 4. Integration Pointers
- **Configuration**: Refer to `references/config.md` for the exact JSON schema of the adaptive plugin.
- **Application Logic**: Refer to `references/hints.md` for how to consume adaptive hints and scheduling guidance in your code.
- **Language Bindings**:
    - Python: `nemo_relay.adaptive`
    - Node.js: `nemo-relay-node/adaptive`
    - Rust: `nemo_relay_adaptive`
