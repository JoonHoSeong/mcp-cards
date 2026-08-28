---
name: nemo-relay-debug-runtime-integration
description: Diagnostic framework for resolving NeMo Relay runtime failures, including native load errors, inactive scopes, missing events, and plugin wiring issues.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# NeMo Relay Runtime Debugging Guide

This skill provides a structured approach to diagnosing why NeMo Relay is not behaving as expected in an application, moving from the lowest (binary) to the highest (logic) layer.

## 1. First-Pass Triage
Before changing code or config, prove which layer is failing:
1. **Load Layer**: Can the binding or native artifact actually load?
2. **Scope Layer**: Is there an active scope when the failing call runs?
3. **Stack Layer**: Is the work happening on the expected scope stack?
4. **Config Layer**: Is the subscriber/plugin configuration actually active?
5. **API Layer**: Is the app using the correct API (Managed vs Manual vs Typed)?

## 2. Troubleshooting Matrix

| Failure Symptom | Likely Root Cause | Diagnostic Action |
|---|---|---|
| **Import/Load Error** | Native extension missing or library path wrong | Rebuild venv (`uv sync`) or check `DYLD_LIBRARY_PATH` (macOS). |
| **Missing Events** | No active scope or registration failed | Verify `get_scope_stack()` is not empty; check if subscriber was registered *before* work. |
| **Wrong Root UUID** | Shared scope stack across requests | Create a fresh `ScopeStack` per independent request. |
| **Middleware Missing** | Incorrect scope ancestry or priority | Check global vs scope-local registration and priority values. |
| **Empty ATIF/ATOF** | Registration order or concurrent agent collision | Ensure registration happens before work; separate agents by root scope. |
| **Payload Conversion Error** | Incompatible provider payload | Implement an explicit `ProviderCodec` for the specific LLM shape. |
| **Plugin Not Acting** | Validation failure or config ignored | Validate config independently; check if the `enabled` flag is `true`. |
| **OTLP/OpenInference Failure** | Version mismatch or endpoint error | Identify Relay version (0.6 vs 0.7); check gRPC endpoint and TLS. |

## 3. Language-Specific Recovery Paths
- **Python**: Rebuild native extensions with `uv sync` from the same environment as the app.
- **Node.js**: Reinstall and rebuild the native addon from the binding package.
- **Go**: Build the release FFI shared library and point the runtime loader (`LD_LIBRARY_PATH`) to the output.
- **Rust**: Run the narrowest core build first, then expand to the binding.

## 4. Diagnostic Checklist
- [ ] Native artifacts are loaded without errors.
- [ ] The call is executed within a valid, non-empty scope.
- [ ] The scope stack is isolated per request.
- [ ] Subscribers are registered before events are emitted.
- [ ] Managed helpers are used instead of only raw callbacks.
- [ ] Provider codecs match the actual payload shape.
- [ ] Plugin configurations are validated and active.
