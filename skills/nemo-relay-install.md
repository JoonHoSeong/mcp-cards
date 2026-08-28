---
name: nemo-relay-install
description: Orchestrate the installation of NeMo Relay CLI, language bindings (Python, Node.js, Rust), or framework integrations (Hermes, OpenClaw, etc.).
license: Apache-2.0
metadata:
  kind: tool
  layer: foundation
  domain: meta
---

# NeMo Relay Installation

This skill handles the initial deployment of NeMo Relay across various target environments. It ensures that the correct package is installed and verified before any runtime configuration begins.

## 1. Installation Path Decision Matrix (HARD RULE)
The agent must select exactly one path based on the user's target outcome. If ambiguous, ask one short clarifying question.

| Path | Target Use Case | Priority | Reference |
| :--- | :--- | :--- | :--- |
| **CLI** | Local gateway, coding-agent (Claude Code/Codex/Hermes) temporary run, or host-plugin setup. | High | `references/cli-install.md` |
| **Integration** | User already uses OpenClaw, Hermes, LangChain, LangGraph, or Deep Agents. | Medium | `references/maintained-integrations.md` |
| **Language PKG** | Direct ownership of tool/model call sites in a Python, Node.js, or Rust app. | Low | `references/language-packages.md` |
| **Source** | Contributors or unpublished changes. | Lowest | Repository Dev Guide |

## 2. Codex Desktop Continuity Gate (CRITICAL SAFETY)
When performing a persistent `nemo-relay install codex` on Codex Desktop:
- **Risk**: A provider-filtering bug can make existing threads appear missing after restart.
- **Required Guardrail**:
  1. Recommend a **temporary transparent run** first.
  2. If persistent install is required, render `assets/codex-desktop-recovery.md` as `NEMO_RELAY_CODEX_DESKTOP_RECOVERY.md` in the workspace root **before** running the installer.
  3. Confirm the recovery-file path is saved before restarting the app.

## 3. Execution Pipeline
1. **Environment Audit**: Inspect OS, arch, existing manifest (`pyproject.toml`, `package.json`, `Cargo.toml`), and venv.
2. **Path Selection**: Apply the Decision Matrix.
3. **Command Preview**: Show the exact install command before execution.
4. **Verification**: Run the specific basic availability check for the chosen path.
5. **Hand-off**:
   - $\rightarrow$ `nemo-relay-get-started` for first-value trial.
   - $\rightarrow$ `nemo-relay-debug-runtime-integration` for config/load issues.

## 4. Verification & Doctor
- Use `nemo-relay doctor` for CLI/Gateway/Agent-readiness issues.
- Use `nemo-relay doctor --plugin <plugin>` specifically for persistent host-plugin verification.
- **Do not** use `doctor` to verify that a language package (pip/npm/cargo) was installed; use the package manager's own list/import checks.

## Related Skills
- [`nemo-relay-get-started`](../nemo-relay-get-started.md) $\rightarrow$ The immediate next step after successful install.
- [`nemo-relay-debug-runtime-integration`](../nemo-relay-debug-runtime-integration.md) $\rightarrow$ Invoked when install succeeds but runtime behavior is missing.
