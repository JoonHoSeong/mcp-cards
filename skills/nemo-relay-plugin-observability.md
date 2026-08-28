---
name: nemo-relay-plugin-observability
description: Guide for configuring NeMo Relay observability via the built-in plugin, subscribers, and exporters (ATOF, ATIF, OpenTelemetry, OpenInference).
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# NeMo Relay Observability Configuration

This skill provides the operational path for wiring observability pipelines in NeMo Relay runtimes to capture and export execution telemetry.

## 1. The Observability Model
NeMo Relay uses a **Canonical Event Stream** architecture:
- **Emission**: Scopes, marks, tool calls, and LLM calls emit events to a single stream.
- **Subscription**: Multiple subscribers can listen to the same stream independently.
- **Transformation**: Exporters translate raw events into specific formats (ATOF $\rightarrow$ ATIF $\rightarrow$ OTLP).

### Subscriber Types
- **Global**: Process-wide persistence.
- **Scope-Local**: Owned by a specific scope; destroyed when the scope closes.
- **Plugin-Installed**: Configuration-driven and reusable.

## 2. Exporter Selection Matrix
Choose the output based on the inspection target:

| Output Format | Best For | Reference |
|---|---|---|
| **ATOF (JSONL)** | Raw lifecycle debugging, offline inspection | `references/atof.md` |
| **ATIF** | Portable execution trajectories, auditing | `references/atif.md` |
| **OpenTelemetry** | OTLP tracing, enterprise monitoring | `references/opentelemetry.md` |
| **OpenInference** | GenAI-specific tracing (standardized) | `references/openinference.md` |

## 3. Version-Specific Configuration (0.6 vs 0.7)
You must identify the Relay version before configuring exporters:

- **NeMo Relay 0.6**: 
    - Uses Observability Config v2.
    - OpenTelemetry and OpenInference are **separate** exporters (`OpenTelemetrySubscriber` vs `OpenInferenceSubscriber`).
- **NeMo Relay 0.7**: 
    - Uses Observability Config v3.
    - A **single typed OpenTelemetry exporter** provides three projections: `full`, `gen_ai`, and `openinference`.

## 4. Operational Workflow
1. **Choose One Output**: Start with **ATOF** as the local proof-of-concept.
2. **Register**: Assign a unique name to the exporter before starting scoped work.
3. **Instrument**: Execute the agent workflow within scopes.
4. **Sanitize**: Verify that sanitization is active *before* production payloads reach external exporters.
5. **Flush & Shutdown**: Follow the version-specific order for deregistration and shutdown.

## 5. Technical Implementation Details

### LLM Annotation Freshness
Relay manages annotation history for LLM calls to ensure context efficiency:
- **Fresh Start**: Every owning agent scope starts fresh.
- **Compaction**: A `compaction` mark refreshes the history.
- **Retention**: The first LLM start after a fresh start retains full history; subsequent starts retain only system instructions, the latest user message, and all following tool/assistant turns.

### Skill Load Events
Tool calls that read a `SKILL.md` automatically emit a `skill.load` mark.
- **Payload**: Contains `skill_name` and metadata about the load source.
- **Inferred Loads**: Ambiguous slash-command expansions emit `skill.load.inferred`.

## 6. Critical Pitfalls
- **Version Mixing**: Never mix 0.6 and 0.7 binding APIs in a single example.
- **Production Leaks**: Never display complete event records while validating an exporter in a production environment.
- **Ordering**: Registering a subscriber *after* the scoped work has started will result in missing telemetry.
