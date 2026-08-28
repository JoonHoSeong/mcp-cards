---
name: jetson-customize-mgbe
description: Enable Jetson Thor 25G/10G/1G MGBE QSFP via kernel-DT overlay.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Jetson Customize MGBE / 25G QSFP

This skill enables high-speed MGBE (Multi-Gigabit Ethernet) connectivity on Jetson Thor custom carriers by pairing BPMP lane allocation with Kernel-DT plumbing.

## 1. The "Overlay-Only" Invariant (HARD RULE)
**This skill MUST NOT edit `ODMDATA`**. 
- All UPHY lane allocations and speed tokens (e.g., `mgbeN-speed-25G`) are owned and emitted by [`jetson-customize-uphy`](../jetson-customize-uphy.md) in its single atomic ODMDATA commit.
- This skill renders the **Kernel-DT overlay** (`status="okay"` + PHY plumbing) to ensure the kernel recognizes the hardware.

## 2. Execution Pipeline
1. **Resolution**: Resolve the active target and Adaptation Guide. Refuse if `custom_carrier_schematic` or `custom_carrier_pinmux_xls` is missing.
2. **Discovery (Question Loop)**: Identify the target MGBE controller, PHY mode (25G/10G/1G), attach kind, I²C bus/addr, and reset GPIOs via `AskUserQuestions`.
3. **BPMP DTB Inspection**: Decompile the BPMP DTB to determine if speed tokens should be top-level (dashed form: `mgbe<N>-speed-25G`) or sub-node form.
4. **Verification**: Run `scripts/pin_verifier.py` for MDC/MDIO/RESET pins. If mismatches are found, route the user to [`jetson-customize-pinmux`](../jetson-customize-pinmux.md).
5. **Overlay Emission**: Append a `/* custom-bsp: mgbe:mgbe<N> */` fragment to the composite custom overlay `.dts`.
6. **Finalization**: Write the `.jetson-customize-mgbe.json` sidecar and drive the downstream next-step chain.

## 3. Critical Gotchas
- **MDIO Plumbing**: When `phy_attach_kind == "phy"`, the `mdio` child node **must** have both `#address-cells = <1>` and `#size-cells = <0>`. Missing either causes the kernel to reject the PHY probe.
- **BPMP-DTB Form Mismatch**: In certain Thor releases, using the sub-node token form (`mgbe<N>_status=disabled`) when the BPMP DTB expects the top-level dashed form causes the **entire ODMDATA line to be dropped at flash time**. Always inspect the BPMP DTB first.
- **L1-L4 Sync**: If the cold boot succeeds but `ip link show` reports `NO-CARRIER`, verify that the UPHY allocation in `jetson-customize-uphy` matches the PHY mode in this skill.

## Related Skills
- [`jetson-customize-uphy`](../jetson-customize-uphy.md) $\rightarrow$ Parent skill that owns UPHY lane allocation and ODMDATA.
- [`jetson-customize-pinmux`](../jetson-customize-pinmux.md) $\rightarrow$ Invoked to fix HSIO pin SFIO mismatches.
- [`jetson-build-source`](../jetson-build-source.md) $\rightarrow$ Compiles the composite overlay into a `.dtbo`.
