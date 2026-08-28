---
name: jetson-quick-start
description: Entrypoint for Jetson / IGX BSP customization; handles disclaimer acceptance, platform selection, and setup mode dispatch.
license: Apache-2.0
metadata:
  kind: dispatcher
  layer: bootstrap
  domain: setup
---

# Jetson Quick Start (BSP Entrypoint)

This skill is the **canonical intake form** for all Jetson BSP customization. It does not perform any actual installation or downloading; it gathers routing-critical answers and dispatches the correct downstream setup skills.

## 1. The Disclaimer Gate (HARD GATE)
Before any setup action, you **must** print the disclaimer and receive explicit acceptance.

**Disclaimer Text:**
```text
================================================================================
DISCLAIMER
These skills help automate Jetson BSP setup and customization, but they do not
replace NVIDIA official documentation or engineering review. Review generated
plans, commands, diffs, and commit messages before accepting them.

Flashing can erase device storage or leave a target temporarily unbootable.
Keep backups and verify the active target, BSP release, and hardware setup
before deploy steps.
================================================================================
```
**Required Action:** Use `AskUserQuestions` with a single choice: `accept_disclaimer` or `cancel`. Stop immediately if not accepted.

## 2. Core Questionnaire (HARD GATE)
Once the disclaimer is accepted, you **must** collect the following four routing-critical fields via `AskUserQuestions`. Do not infer these from existing profiles or filenames.

| Field | Description | Valid Options / Source |
| :--- | :--- | :--- |
| `mode` | Setup Path | `Auto Setup`, `Guided Setup`, `Use Existing Workspace`, `cancel` |
| `active_platform` | Hardware Target | Parsed from `bsp-platforms-catalogue.md` (Index-based) |
| `bsp_release` | Software Version | Concrete token (e.g., `R36.4.4`, `38.2.1`) from official archive or `skip` |
| `custom_carrier` | Hardware Mod | `no_custom_carrier`, `add_custom_carrier`, `keep_existing`, `skip` |

## 3. Setup Mode Dispatch Logic
Based on the submitted `mode`, invoke the following downstream skills in order:

### Path A: `Auto Setup` (Fresh Start)
`jetson-set-target` $\rightarrow$ `jetson-download-bsp` $\rightarrow$ `jetson-init-image` $\rightarrow$ `jetson-init-source` $\rightarrow$ `jetson-link-docs` $\rightarrow$ `jetson-generate-kb` $\rightarrow$ (optional) `jetson-derive-carrier`.

### Path B: `Guided Setup` (Partial Existing)
`jetson-set-target` $\rightarrow$ `jetson-init-image` $\rightarrow$ `jetson-init-source` $\rightarrow$ `jetson-link-docs` $\rightarrow$ `jetson-generate-kb` $\rightarrow$ (optional) `jetson-derive-carrier`.

### Path C: `Use Existing Workspace` (Verification Only)
1. **Verify**: Check for `Linux_for_Tegra/` and `rootfs/etc/nv_tegra_release`.
2. **Dispatch**: `jetson-set-target` $\rightarrow$ `jetson-generate-kb` $\rightarrow$ (optional) `jetson-derive-carrier`.
3. Route to `init` skills only if prerequisites are missing.

## 4. Next Steps: I/O Customization
After dispatch, if `documents.carrier_board_spec` or similar slots are bound, suggest these tools:
- `jetson-customize-pinmux` $\rightarrow$ `jetson-customize-uphy` $\rightarrow$ `jetson-customize-pcie` / `usb` / `mgbe` $\rightarrow$ `jetson-customize-camera`.

## Related Skills
- [`jetson-download-bsp`](../jetson-download-bsp.md) $\rightarrow$ The actual fetcher for Auto Setup.
- [`jetson-init-image`](../jetson-init-image.md) $\rightarrow$ Image materialization.
- [`jetson-init-source`](../jetson-init-source.md) $\rightarrow$ Source tree preparation.
- [`jetson-derive-carrier`](../jetson-derive-carrier.md) $\rightarrow$ Custom flash config generation.
