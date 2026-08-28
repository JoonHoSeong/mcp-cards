---
name: nemo-relay-instrument-calls
description: Guide for wrapping tool and LLM/provider call sites with NeMo Relay scopes and managed execution APIs to capture lifecycle events, middleware, and guardrails.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Tool and LLM Instrumentation Guide

This skill provides the operational framework for integrating existing tool functions and LLM provider calls into the NeMo Relay runtime.

## 1. Instrumentation Fundamentals
The goal is to wrap original callable behavior with Relay's lifecycle capture without altering the business logic.

### The Execution Hierarchy
1. **Scope Boundary**: Every instrumented call must occur within a scope (e.g., `Agent`, `Request`, `Workflow`).
2. **Conditional Guardrails**: Runs first on raw input. If rejected, a standalone mark event is emitted, and execution stops.
3. **Request Intercepts**: Runs second to rewrite the real input reaching the callback.
4. **Execution Intercepts**: Wraps the callback in a middleware `next` pattern; can short-circuit by returning a custom result.
5. **The Callback**: The original tool or LLM provider function runs.
6. **Sanitize-Response Guardrails**: Affects the emitted end-event payload only; does not change the value returned to the app.

## 2. Implementation Paths

### Path A: Managed Execution APIs (Recommended)
Use these high-level helpers to ensure all lifecycle events and middleware are triggered correctly.
- **Python**: `tools.execute(...)`, `llm.execute(...)`
- **Node.js**: `toolCallExecute(...)`, `llmCallExecute(...)`
- **Rust**: `tool_call_execute(...)`, `llm_call_execute(...)`
- **Go**: `tools.Execute(...)`, `llm.Execute(...)`

### Path B: Manual Lifecycle APIs
Use only when the host framework owns execution and cannot be wrapped by managed helpers.
- **Requirement**: Every `start` call must have a matching `end` or `error` path with explicit semantic payloads.
- **Risk**: Failure to balance start/end calls leads to "hanging" scopes and corrupted telemetry.

## 3. Key Operational Rules
- **Separation of Concerns**: Let the original callable handle business logic; let NeMo Relay handle lifecycle events, middleware, and metadata.
- **Streaming LLMs**: Wrappers collect chunks and finalize the response at stream end. Dropping a stream early may prevent subscribers from seeing the complete output.
- **Context Propagation**: Ensure the scope stack is propagated if the call hops threads or async tasks (see `nemo-relay-instrument-context-isolation`).

## 4. Validation Checklist
- [ ] Scope boundary is established before the first call.
- [ ] Tool functions are wrapped without losing original arguments/results.
- [ ] LLM calls are wrapped at the appropriate abstraction layer.
- [ ] Metadata (model name, call identifiers) is attached for trace diagnostics.
- [ ] Context propagation is handled for async/multi-threaded boundaries.
