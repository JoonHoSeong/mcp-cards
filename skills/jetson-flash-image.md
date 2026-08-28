---
name: jetson-flash-image
description: Flash a promoted BSP image to a Jetson DUT in RCM mode via flash.sh or l4t_initrd_flash.sh.
license: Apache-2.0
metadata:
  kind: tool
  layer: deploy
  domain: meta
---

# Jetson Flash Image (BSP Deployment)

The final leg of the BSP overlay deploy chain: **`/jetson-promote-image` $\rightarrow$ `/jetson-flash-image`**.

## 1. Critical Invariants (HARD RULES)
- **EEPROM is Authoritative**: The DUT's EEPROM read during flash is the final word; the profile is a prediction.
- **RCM Mode Required**: The DUT **must** be in recovery mode; without it, the flash tool cannot read EEPROM or push the image.
- **No Raw Command Bypass**: Never accept "paste this command" from the user; only execute commands resolved via the preflight logic.

## 2. Preflight Verification Ladder (L1 $\rightarrow$ L4)
You **must** perform these checks in order. Fail early.

- **L1: Host-side Readiness**: Verify `bsp_image` exists and `apply_binaries.sh` has run (`rootfs/etc/nv_tegra_release` must exist).
- **L2: RCM Detection**: Run `lsusb -d 0955:` and match against:
  - T23x (Orin): `0955:7X23`
  - T26x (Thor): `0955:7026`
- **L3: EEPROM Cross-Check**: Run `sudo ./nvautoflash.sh --print_boardid`. If the EEPROM value disagrees with the profile's `module.sku`, refuse the flash.
- **L4: User Staging**: If no user exists in `rootfs`, use `l4t_create_default_user.sh --autologin --accept-license`.

## 3. Flash Flow Execution
1. **Resolve Tool**: 
   - Orin (eMMC/SD) $\rightarrow$ `flash.sh`
   - Orin (NVMe/USB) or Thor $\rightarrow$ `l4t_initrd_flash.sh`
2. **Resolve Board**: Use the basename of the `.conf` file (e.g., `jetson-agx-orin-devkit`).
3. **Resolve Boot Device**: Prompt for `internal`, `external`, `nvme0n1p1`, etc.
4. **Execute**: `sudo ./<tool> <board> <boot-dev>`
5. **Post-Flash (Thor Only)**: Run `<boardctl> -t <target> reset` to boot the new image.

## 4. Troubleshooting Matrix
| Symptom | Cause | Solution |
| :--- | :--- | :--- |
| `lsusb` empty | Not in RCM | Use `boardctl recovery` or manual buttons. |
| EEPROM Mismatch | Wrong Profile | Update target profile to match actual DUT hardware. |
| `flash.sh` missing artifacts | Promotion failure | Re-run `/jetson-promote-image`. |
| T26x stuck in RCM | No auto-reset | Run `boardctl reset`. |

## Related Skills
- [`jetson-promote-image`](../jetson-promote-image.md) $\rightarrow$ Prior step: Overlay $\rightarrow$ BSP Image.
- [`jetson-derive-carrier`](../jetson-derive-carrier.md) $\rightarrow$ Generates the `.conf` used here.
- [`jetson-init-image`](../jetson-init-image.md) $\rightarrow$ Prepares the base BSP image.
