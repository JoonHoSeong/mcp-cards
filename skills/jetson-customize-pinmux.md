---
name: jetson-customize-pinmux
description: Per-pin SFIO / direction / initial-state configurator for Jetson Orin/Thor custom carriers from pinmux XLSM.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Jetson Customize Pinmux (Per-Pin SFIO/State)

This skill manages the mapping of SoC CVM balls to specific functions (SFIO), directions, and electrical states using the pinmux XLSM as the ground truth.

## 1. The "No Kernel-DT" Invariant (HARD RULE)
Unlike other I/O skills, **pinmux has no kernel-DT overlay surface**. It operates exclusively at the BCT (Boot Configuration Table) level.
- **Source of Truth**: Pinmux XLSM $\rightarrow$ `modify_pinmux.py`.
- **Target**: Three BCT DTSIs (`pinmux`, `gpio`, `padvoltage`) in the overlay tracker.
- **Flash-time Integration**: The carrier conf's `PINMUX_CONFIG=`, `GPIOINT_CONFIG=`, and `PMC_CONFIG=` lines point to these files.

## 2. Execution Pipeline (Steps 1-8)
1. **XLSM Resolution**: Resolve the pinmux `.xlsm` from `documents.custom_carrier_pinmux_xls` or `documents.ref_devkit_pinmux_xls`.
2. **Probe**: Run `scripts/modify_pinmux.py probe` to parse the XLSM into a per-carrier pinmap JSON.
3. **Lookup**: Resolve user queries (CVM ball, signal name) to surface supported SFIOs and `configurable: yes/no` status.
4. **Set-Pin (Hard Gate)**: Use `AskUserQuestions` for:
   - Q1-Q3: `sfio`, `direction`, `initial_state` (Always asked).
   - Q4-Q6: `pull`, `drive_type`, `open_drain` (Asked **only** if `configurable: yes`).
5. **Generate**: Run `scripts/modify_pinmux.py generate` to emit the three BCT DTSIs into `bootloader/`.
6. **Wrapper Update**: Edit the `#include` lines in the `.dts` wrappers (in `bootloader/generic/BCT/`) to point to the new bare basenames of the generated `.dtsi` files.
7. **Commit**: Group the three DTSIs and the wrapper edits into a **single customization commit**.
8. **Sidecar**: Write the `.jetson-customize-pinmux.json` run-state sidecar.

## 3. Critical Gotchas
- **Fixed-Function Pads**: Pads like `LP5XA_*` or `UPHYDS_*` are fixed-function; the skill **must skip Q4-Q6** for these to avoid confusing the user.
- **GPIO Capability**: If `sfio=gpio` is chosen, the pin must have a valid `gpio=GPIOn_PD.NN` entry in the XLSM; otherwise, reject the call.
- **Marker Idempotency**: Use `// custom-bsp: pinmux` markers to ensure that re-running `generate` updates existing edits rather than duplicating them.
- **Bare Basenames**: In the wrapper `#include`, use the bare filename (e.g., `#include "tegra234-mb1-bct-pinmux-custom.dtsi"`) as the BCT build's `-I bootloader/` handles resolution.

## Related Skills
- [`jetson-derive-carrier`](../jetson-derive-carrier.md) $\rightarrow$ Must run first to create the BCT DTSI forks and update the carrier conf.
- [`jetson-init-source`](../jetson-init-source.md) $\rightarrow$ Provides the overlay tracker for commits.
- [`jetson-customize-uphy`](../jetson-customize-uphy.md) $\rightarrow$ Often invoked when an HSIO pin mismatch is found during UPHY allocation.
