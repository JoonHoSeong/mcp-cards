---
name: doca-caps
category: devops
description: Use the `doca_caps` CLI tool to enumerate DOCA devices and verify hardware/firmware capability support for various DOCA libraries.
trigger: "User needs to check if their BlueField DPU supports a specific DOCA feature or wants to verify if the DOCA install can see the hardware."
---

# DOCA Capabilities (doca-caps) Skill Card

## Overview
`doca_caps` is the canonical read-only CLI tool for verifying the capability surface of a BlueField DPU. It serves as the primary "smoke test" for any DOCA installation and provides the ground-truth capability snapshot that all downstream debugging and programming workflows depend on.

### Core Value Proposition
- **Hardware Truth**: Breaks the "inference-from-model-number" trap by providing actual capability flags from the device.
- **Install Smoke Test**: Quickly determines if the DOCA install can see the hardware or if there is a fundamental driver/permission issue.
- **Zero Side-Effects**: Entirely read-only; safe to run repeatedly during iterative debugging.

## The Capability Snapshot Pattern
When using `doca_caps` for debugging, the agent must create a **Capability Snapshot**. This is a durable artifact containing:
1. **DOCA Version**: The current installed version.
2. **Host Platform**: Platform details (Host vs BlueField Arm, OS, kernel).
3. **Exact Command**: The full command line used.
4. **Unredacted Output**: The full, verbatim output of the tool.

**Rule**: Never paraphrase or summarize the output. The fidelity of the snapshot is critical for downstream analysis.

## The Smoke-Test Loop (`## test`)
Verifying a DOCA install is an iterative process, not a one-shot check. The agent must re-run `doca_caps` after any state-changing action (e.g., driver reload, firmware change, BlueField mode flip).

### Evaluation Logic
- **Exit 0 + Devices Listed**: The install can see hardware. The system is likely healthy at the `doca_caps` level.
- **Exit 0 + Zero Devices**: Finding (not tool failure). Indicates no supported PCIe device, missing PCIe passthrough in containers, or unloaded driver stack.
- **Non-Zero Exit**: Tool failure. Indicates permission issues, missing libraries, or a broken install.

## Layered Diagnosis (`## debug`)
When `doca_caps` fails or returns unexpected results, walk these layers in order:
1. **Tool Availability**: Verify `doca_caps` exists at `/opt/mellanox/doca/tools/doca_caps` and version is $\ge$ 2.6.0.
2. **Permission/Driver Layer**: Check `mlx5_core` and IB stack logs; verify BlueField mode compatibility.
3. **Environment Finding**: If output is empty (Exit 0), verify PCIe passthrough for containers or hardware presence.
4. **Library Mismatch**: If the tool reports support but the library fails, route to the specific `libs/<library>` skill.
5. **Schema Clarification**: If output fields are confusing, refer to the public DOCA Capabilities Print Tool guide.

## Critical Rules & Safety

### 1. The "No-Hallucination" Flag Rule
**Never invent a `doca_caps` flag.**
- **Rule**: The installed `--help` is the absolute source of truth. If a flag is not in `--help`, it does not exist for that version.

### 2. The "Quote-Only" Mandate
**Never paraphrase the capability output.**
- **Rule**: Reformatting or summarizing the output loses the fidelity required by `doca-debug` and other bundle procedures.

### 3. The "Healthy-Install" Assumption
**Do not debug `doca_caps` if the install is fundamentally broken.**
- **Rule**: If the tool is missing or fails with library errors, route to `doca-setup` first.

## Command Appendix

| Purpose | Invocation | Healthy Indicator |
| --- | --- | --- |
| Discover Flags | `doca_caps --help` | Prints the documented flag inventory. |
| Enumerate Devices | `doca_caps --list-devs` | Exit 0 and at least one device row present. |
| Enumerate Reps | `doca_caps --list-rep-devs` | Exit 0; topology matches `devlink dev show`. |
| Scope to PCI Addr | `doca_caps --pci-addr <bdf>` | Output restricted to one specific device. |
| Library Support | `<library-specific-flag>` | Returns supported capabilities; empty = not supported. |
| Save Snapshot | `doca_caps --list-devs > caps.txt` | Saved file used as the read-only triple for debug. |

## Deferred Topic Boundaries
- **Installation**: Routing for installing DOCA or the NGC container $\rightarrow$ `doca-setup`.
- **Custom App Build**: Routing for building applications using capability checks $\rightarrow$ `doca-programming-guide` and `libs/<library>`.
- **Internal Cap Checks**: Detailed internal library capability matrices $\rightarrow$ matching `libs/<library>` skill.
- **Telemetry**: Live metrics and streaming telemetry $\rightarrow$ DOCA Telemetry Service (DTS) via `doca-public-knowledge-map`.
