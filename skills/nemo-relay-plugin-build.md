---
name: nemo-relay-plugin-build
description: Framework for building and packaging reusable NeMo Relay runtime behavior as configuration-activated plugins with deterministic validation and rollback-safe registration.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# NeMo Relay Plugin Development Framework

This skill defines the standard for packaging reusable runtime behavior (subscribers, guardrails, intercepts) as configuration-activated plugins.

## 1. The Plugin Model
A plugin is a reusable process-level component identified by a stable `kind` string. It decouples business logic from application startup code.

### Core Lifecycle
1. **Validation**: Deterministic and side-effect free. Inspects the JSON config and returns diagnostics *before* any runtime changes occur.
2. **Registration**: Installs behavior through a `PluginContext`. This ensures that the system can qualify runtime names and perform an atomic rollback if registration fails.
3. **Activation**: Controlled via the `enabled` flag in the shared plugin document.

## 2. Implementation Workflow

### Step 1: Surface Selection
Determine which NeMo Relay surface the plugin will target:
- **Subscribers**: Exporting events.
- **Guardrails**: Sanitize or conditionally block requests.
- **Intercepts**: Request intercepts, execution intercepts, or stream execution intercepts.

### Step 2: Configuration Design
- **JSON Compatibility**: Config must be JSON-compatible across Rust, Python, and Node.js.
- **Naming**: Use `snake_case` for all configuration keys.
- **No Live Objects**: Never put callables, client instances, credentials, or file handles directly in the config. Use references/IDs instead.

### Step 3: Validation Logic
Validation must be "pure":
- **No Network/IO**: Do not open connections or read files during validation.
- **No State Mutation**: Do not register middleware or mutate process state during validation.
- **Comprehensive Diagnostics**: Report missing fields, unsupported values, and invalid combinations.

### Step 4: Registration via PluginContext
Behavior must be installed through the `PluginContext` rather than global registration calls. This allows the plugin system to manage ownership and cleanup.

## 3. Configuration Schema
Plugins are managed via a top-level document:
```json
{
  "version": 1,
  "components": [
    {
      "kind": "your-plugin-kind",
      "enabled": true,
      "config": { "param": "value" }
    }
  ],
  "policy": {
    "unknown_component": "warn",
    "unknown_field": "warn",
    "unsupported_value": "error"
  }
}
```
*Note: Dynamic plugins can declare a `config_schema` in `relay-plugin.toml` to enable structured editing in `nemo relay plugins edit`.*

## 4. Critical Pitfalls & Safety
- **Validation Skip**: Do not skip validation for disabled components; operators should find config errors before rollout.
- **Partial Activation**: Ensure that a registration failure triggers a complete rollback of any partial behavior installed.
- **Telemetry Leaks**: Never export raw production payloads or secrets. Always apply sanitization before data leaves the process.
- **Logic in Config**: Keep business logic in the plugin code; the config should only contain parameters.

## 5. Binding Pointers
- **Python**: `nemo_relay.plugin`
- **Node.js**: `nemo-relay-node/plugin`
- **Rust**: `nemo_relay::plugin`
