---
name: nemo-relay-get-started
description: Guide first-time NeMo Relay users through the least-complex trial path to verify observable value.
license: Apache-2.0
metadata:
  kind: guide
  layer: foundation
  domain: meta
---

# Get Started with NeMo Relay

This skill guides new users to their first "observable success" with the least friction. The goal is to prove the value of instrumentation before moving to production setup.

## 1. Try-Now Path Selection (Priority Order)
The agent must select the first path that fits the user's environment.

| Path | Trigger | Action | Reference |
| :--- | :--- | :--- | :--- |
| **CLI Try-Now** | Generic "try" request or no existing app code. | Wrap Codex/Claude Code/Hermes via CLI. | `references/cli-try-now.md` |
| **Integration** | User uses LangChain, LangGraph, OpenClaw, or Hermes. | Use maintained framework integration. | `references/built-in-integrations-try-now.md` |
| **Manual Lang** | Python/Node.js/Rust app owns call sites directly. | Manual instrumentation. | `references/manual-language-try-now.md` |

## 2. The "First-Value" Contract (HARD RULE)
To ensure the trial is a success, the agent must:
1. **Identify Boundary**: Explicitly explain where Relay attaches (e.g., "wrapping the CLI").
2. **Minimal Change**: Show the exact minimal code/command change required.
3. **Observable Proof**: Do not treat a successful LLM response as proof. **Verify evidence** (e.g., captured events in the Observability plugin or logs).
4. **Non-Sensitive Trial**: Use read-only, non-sensitive tools/prompts for the first run.

## 3. Progression Logic
Once the first value is proven, the agent must suggest **exactly one** plugin to demonstrate extensibility without rewriting code:
- **Observability**: If not already used for proof.
- **Adaptive**: For runtime behavior optimization.
- **Guardrails**: For policy checks.
- **Model Pricing**: For cost estimation.

## 4. Handoffs
- $\rightarrow$ `nemo-relay-instrument-calls` for application expansion.
- $\rightarrow$ `nemo-relay-plugin-observability` for durable observability.
- $\rightarrow$ `nemo-relay-debug-runtime-integration` if the trial fails (e.g., missing events).

## Related Skills
- [`nemo-relay-install`](../nemo-relay-install.md) $\rightarrow$ Prerequisite. If install is missing, route here first.
- [`nemo-relay-debug-runtime-integration`](../nemo-relay-debug-runtime-integration.md) $\rightarrow$ If trial fails.
