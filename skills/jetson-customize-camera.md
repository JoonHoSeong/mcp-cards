---
name: jetson-customize-camera
description: Enable MIPI/GMSL camera sensors on Jetson Thor/Orin custom carriers via kernel-DT overlay.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Jetson Customize Camera (CSI/MIPI/GMSL)

This skill enables camera sensor bring-up on Jetson custom carriers by rendering a kernel-DT overlay from in-tree sensor definitions.

## 1. The "No ODMDATA" Invariant (HARD RULE)
Unlike PCIe or MGBE, **cameras do not consume UPHY lanes**. They use a separate CSI PHY pool.
- **Action**: This skill emits **only** a kernel-DT overlay.
- **Constraint**: The carrier conf's `ODMDATA` line remains untouched.

## 2. Execution Pipeline
1. **Sensor Discovery**: Glob in-tree per-platform camera dtbos to build a list of supported sensors.
2. **Support Check**: Cross-reference the chosen sensor against the Camera Development Guide and carrier schematic.
3. **Wiring Capture**: Use the in-tree `.dtsi` as the source of truth for wiring. For custom sensors, capture wiring from the user.
4. **Overlay Generation**:
   - Cpp-expand the in-tree DTSI.
   - Extract the `fragment@N` body.
   - Append to the composite custom overlay `.dts` with the marker `/* custom-bsp: camera:<sensor> */`.
5. **Header Update**: Idempotently set the `jetson-header-name` on the composite root.
6. **Pin Verification**: Run `scripts/pin_verifier.py` for I²C, MCLK, and Reset/PWDN pins. Route mismatches to [`jetson-customize-pinmux`](../jetson-customize-pinmux.md).

## 3. Critical Gotchas
- **The Dual-Fragment Trap**: Adding more than one `fragment@N` for the same sensor (e.g., adding a manual override on top of an in-tree copy) creates duplicate sibling subtrees, causing the kernel to silently drop the deep tree. **Always use a single, merged fragment.**
- **Stub Overlay Footgun**: Creating a "stub" overlay (just setting `status="okay"`) without the full sensor body/mode tables results in `all channel init failed` errors. **Always splice the full sensor body.**
- **Compatible Mismatch**: If the composite root `compatible` string does not intersect with the live DT compatible, the UEFI plugin-manager will silently skip the overlay.
- **Header-Only Registration**: Do NOT append the in-tree `.dtbo` to `OVERLAY_DTB_FILE`. Only register the rendered composite overlay.

## Related Skills
- [`jetson-customize-pinmux`](../jetson-customize-pinmux.md) $\rightarrow$ Invoked to fix I²C/MCLK/Reset pin SFIOs.
- [`jetson-build-source`](../jetson-build-source.md) $\rightarrow$ Compiles the composite overlay into a `.dtbo`.
- [`jetson-derive-carrier`](../jetson-derive-carrier.md) $\rightarrow$ Produces the base overlay that this skill stacks upon.
