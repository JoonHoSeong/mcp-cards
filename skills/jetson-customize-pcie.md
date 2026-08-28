---
name: jetson-customize-pcie
description: Configure per-controller PCIe status, lanes, and link-speed for Jetson Orin/Thor custom carriers.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# Jetson Customize PCIe (Controller-level)

This skill manages the Kernel-DT overlay for PCIe controllers on Jetson custom carriers. It is a **sub-skill** of `jetson-customize-uphy`.

## 1. The "Overlay-Only" Invariant (HARD RULE)
**This skill MUST NOT edit `ODMDATA`**. 
- All power-rail, RefCLK, and UPHY-lane-allocation tokens (e.g., `pcie@N_status=okay`, `uphy0-config-N`) are owned and emitted by [`jetson-customize-uphy`](../jetson-customize-uphy.md) in a single atomic commit.
- This skill only translates that allocation into a **Kernel-DT overlay fragment** (`status="okay"`, `num-lanes`, `max-link-speed`).

## 2. Execution Pipeline
1. **Resolution**: Resolve the `uphy-state` JSON sidecar from `jetson-customize-uphy`. Refuse the run if the sidecar is missing.
2. **Topology Diff**: Decompile the reference DTB and grep the carrier schematic for `PEX<N>_*` nets to identify routed controllers.
3. **Verification**: Run `scripts/pin_verifier.py` to check `PE<N>_CLKREQ_L` and `PE<N>_RST_L` signals.
4. **Plan Derivation**: 
   - `enable` $\rightarrow$ derived from `uphy-state` (Allocated $\rightarrow$ `okay`, Not-Allocated $\rightarrow$ `disabled`).
   - `lanes/speed` $\rightarrow$ derived from Adaptation Guide.
5. **Overlay Emission**: Append `fragment@N` blocks to the composite custom overlay `.dts` in `bsp_sources/`.
6. **Consistency Check**: Cross-check the resulting overlay against the previously committed ODMDATA. If they disagree, stop and ask the user for recovery (Do **not** use `git reset --hard` autonomously).

## 3. Critical Gotchas
- **RC Pinning**: Mode is hard-pinned to `rc` (Root Complex). Endpoint (EP) mode requires an explicit `mode_override="ep"` flag.
- **Lanes vs. UPHY**: The `num-lanes` property in the kernel overlay must strictly match the lane count allocated in the UPHY configuration.
- **Address Sourcing**: Always source the PCIe node address (e.g., `pcie@C0`) from the reference DTB, never from hard-coded tables.

## Related Skills
- [`jetson-customize-uphy`](../jetson-customize-uphy.md) $\rightarrow$ Parent skill that owns UPHY allocation and ODMDATA.
- [`jetson-customize-pinmux`](../jetson-customize-pinmux.md) $\rightarrow$ Invoked to fix HSIO pin mismatches found during verification.
- [`jetson-build-source`](../jetson-build-source.md) $\rightarrow$ Compiles the composite overlay into a `.dtbo`.
