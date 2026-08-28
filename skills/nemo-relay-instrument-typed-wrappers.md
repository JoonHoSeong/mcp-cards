---
name: nemo-relay-instrument-typed-wrappers
description: Guide for implementing typed wrappers and provider codecs to enable strong domain typing while preserving JSON middleware interoperability in NeMo Relay.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# NeMo Relay Typed Wrappers & Codecs Guide

This skill provides the framework for using strong domain types in tool and LLM integrations without breaking the underlying JSON-based middleware and event system.

## 1. The Codec Model
A **Typed Value Codec** acts as a pure boundary translator. It converts application-facing objects to JSON before the Relay runtime processes them, and converts JSON back to objects for the caller.

### Translation Flow
`App Object` $\rightarrow$ `Codec.encode()` $\rightarrow$ `JSON` $\rightarrow$ `Middleware/Events` $\rightarrow$ `Codec.decode()` $\rightarrow$ `App Object`

## 2. Implementation Options

### Value Codecs (Application Domain)
- **`JsonPassthrough`**: For JSON-native values.
- **`DataclassCodec` / `PydanticCodec` (Python)**: For existing stable domain models.
- **Custom Codecs**: For domain-specific wire shapes.
- **`BestEffortAnyCodec`**: Use only as a last resort when schemas are unavailable.

### Provider Codecs (LLM Specific)
These normalize provider-specific payloads (OpenAI, Anthropic, etc.) so middleware can inspect messages, model names, and usage without knowing the provider's raw API.
- **Request Codecs**: Run before intercepts. They merge annotated edits back into the provider-shaped request.
- **Response Codecs**: Annotate end-events with `id`, `model`, `usage`, and `finish_reason` without modifying the raw response returned to the app.

## 3. Operational Constraints
- **JSON Interop**: Middleware and Guardrails *always* see the serialized JSON, not the typed objects.
- **Preservation**: Provider codecs must preserve any fields they do not understand to avoid data loss.
- **Failure Safety**: A response codec failure must not break the underlying LLM call.

## 4. Integration Checklist
- [ ] Codec output is strictly JSON-compatible.
- [ ] Required domain fields survive the `encode` $\rightarrow$ `decode` cycle.
- [ ] Middleware sees the expected serialized shape for its logic.
- [ ] Provider codecs match the actual payload shape of the target LLM.
- [ ] `BestEffortAnyCodec` is avoided in favor of explicit schemas where possible.
