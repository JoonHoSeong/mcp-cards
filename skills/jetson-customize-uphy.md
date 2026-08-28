---
name: jetson-customize-uphy
description: Configure Jetson UPHY lane allocation (uphy0/uphy1-config) on Orin/Thor custom carriers.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Jetson Customize UPHY Lane Allocation

This skill manages the shared high-speed PHY pool (UPHY) on Tegra234 (Orin) and Tegra264 (Thor) to remap PCIe, MGBE, USB3, or UFS controllers.

## 1. The Two-Surface Invariant (HARD RULE)
UPHY configuration requires atomic synchronization across two distinct surfaces. **This skill owns the ODMDATA surface; sub-skills own the Kernel-DT surface.**

| Surface | Target | Role | Owner |
| :--- | :--- | :--- | :--- |
| **ODMDATA** | BPMP DTB | UPHY lane power, RefCLK gating, Controller power rails | `jetson-customize-uphy` |
| **Kernel-DT** | Kernel DTB | Kernel probe, Lane width, Link speed | `jetson-customize-pcie` / `mgbe` / `usb` |

**Failure to sync both results in either a BL31 SError reboot loop (ODMDATA missing) or a kernel probe timeout (Kernel-DT missing).**

## 2. Execution Pipeline
1. **Resolution**: Resolve Adaptation Guide, schematic, and TRM. Refuse if `custom_carrier_schematic` or `custom_carrier_pinmux_xls` is missing.
2. **Lane Discovery**: Cross-reference schematic net names (e.g., `PEX5_LN0+-`) against documented `uphyX-config-N` indices in the Adaptation Guide.
3. **Selection (Hard Gate)**: Use `AskUserQuestions` to select the UPHY config for each surface (uphy0, uphy1).
4. **ODMDATA Atomic Commit**:
   - Emit `uphyX-config-N` tokens.
   - Emit per-controller tokens (e.g., `pcie@N_status=okay`, `mgbeN-speed-del`).
   - **Crucial**: Emit `UPHY_CONFIG=""` clear if `uphy0-config-6` is selected.
5. **Kernel-DT Dispatch**: Generate a per-controller allocation table and invoke the sub-skills:
   - $\rightarrow$ `jetson-customize-pcie`
   - $\rightarrow$ `jetson-customize-mgbe`
   - $\rightarrow$ `jetson-customize-usb`
6. **Final Sync**: Perform a consistency check between the committed ODMDATA and the resulting Kernel-DT overlays.

## 3. Critical Gotchas
- **MGBE-Speed-Del**: When disabling an MGBE controller, you **must** emit the `mgbeN-speed-del` token. Failing to do so arms FMON on the controller's clocks, leading to a BL31 SError reboot loop.
- **BPMP-DTB vs Kernel-DT**: If a controller doesn't enumerate, check if `status="okay"` exists in the Kernel-DT but `pcie@N_status=okay` is missing from ODMDATA.
- **T264 (Thor) Surface**: Thor has two UPHY surfaces (0 and 1); both must be configured explicitly.

## Related Skills
- [`jetson-customize-pcie`](../jetson-customize-pcie.md) $\rightarrow$ Sub-skill for PCIe Kernel-DT overlays.
- [`jetson-customize-mgbe`](../jetson-customize-mgbe.md) $\rightarrow$ Sub-skill for MGBE Kernel-DT overlays.
- [`jetson-customize-usb`](../jetson-customize-usb.md) $\rightarrow$ Sub-skill for USB3 Kernel-DT overlays.
- [`jetson-derive-carrier`](../jetson-derive-carrier.md) $\rightarrow$ Produces the carrier conf edited by this skill.
