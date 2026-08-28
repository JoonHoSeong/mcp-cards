---
name: jetson-customize-usb
description: Enable/disable Jetson USB2/USB3 SS ports via kernel-DT overlay.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Jetson Customize USB (Per-Port Control)

This skill manages the enablement and role configuration of USB2 and USB3 SuperSpeed (SS) ports on Jetson Orin/Thor custom carriers.

## 1. The "Three-Place Lockstep" Invariant (HARD RULE)
USB configuration on Tegra is fragile. To enable or disable a port without crashing the entire xHCI host probe, changes **MUST** be applied in three places in lockstep:

| Place | Target | Action |
| :--- | :--- | :--- |
| **1. Lane (PHY)** | `xusb_padctl/pads/usb<2\|3>/lanes/usb<2\|3>-N` | Set `status="disabled"` to stop the PHY from providing a lane. |
| **2. Port (Binding)** | `xusb_padctl/ports/usb<2\|3>-N` | Set `status="disabled"` to remove the port from the topology. |
| **3. Host xHCI List** | `bus@0/usb@<addr>.phys` & `.phy-names` | **Remove** the phandle/name of the disabled PHY from the array. |

**Failure to follow this lockstep (e.g., disabling the port but leaving the PHY in the xHCI list) results in an empty `lsusb` for ALL ports.**

## 2. Execution Pipeline
1. **Topology Discovery**: Build the USB companion graph (USB3 SS $\rightarrow$ USB2) from the in-tree DTB.
2. **Selection**: Use `AskUserQuestions` to determine which ports to flip and their desired roles (host/device/otg).
3. **Verification**: Run `scripts/pin_verifier.py` for VBUS-EN, OC, and CC GPIOs. Route SFIO mismatches to [`jetson-customize-pinmux`](../jetson-customize-pinmux.md).
4. **Overlay Emission**: Append fragments for `usb:padctl` and `usb:xhci` to the composite custom overlay `.dts`.
5. **Companion Cascade**: If a USB2 port is disabled, the skill **must** automatically disable its associated USB3 SS companion to prevent `tegra-xusb: failed to enable PHYs: -19` errors.
6. **Finalization**: Write the `.jetson-customize-usb.json` sidecar.

## 3. Critical Gotchas
- **The "Empty lsusb" Trap**: If all USB ports disappear, the most likely cause is a broken three-place lockstep (usually a missing entry removal from the xHCI `phys` list).
- **UPHY Sync**: For USB3 SS ports, the skill must verify that the lanes are allocated in the `jetson-customize-uphy` sidecar before enabling the port.
- **OTG Restriction**: Only `usb2-0` is OTG-capable. Any attempt to set `dr_mode="otg"` on other ports must be rejected.

## Related Skills
- [`jetson-customize-uphy`](../jetson-customize-uphy.md) $\rightarrow$ Parent skill that owns UPHY lane allocation.
- [`jetson-customize-pinmux`](../jetson-customize-mux.md) $\rightarrow$ Invoked to fix VBUS-EN/OC/CC pin SFIOs.
- [`jetson-build-source`](../jetson-build-source.md) $\rightarrow$ Compiles the composite overlay.
