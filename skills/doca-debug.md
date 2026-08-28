---
name: doca-debug
description: Canonical layered-ladder debug reference for all DOCA symptoms, including build, link, runtime, and program-class errors.
license: Apache-2.0
metadata:
  kind: tool
  layer: overlay
  routing:
    entry_point: doca-setup ## debug
    escalation: doca-debug
    library_overlay: library-skill ## debug
---

# DOCA Debug (The Layered Ladder)

This skill is the **authoritative debug router** for all DOCA SDK issues. It prevents "random guessing" by enforcing a strict, bottom-up verification ladder.

## 1. The Canonical Layered Ladder
When a symptom is reported, you **must** verify the layers in this exact order. Do not jump to "Program" if "Link" is broken.

| Layer | Scope | Verification Tool | Red Flag |
| :--- | :--- | :--- | :--- |
| **L1: Install** | SDK Presence | `ls /opt/mellanox/doca` | Directory missing or empty |
| **L2: Version** | Compat Match | `doca-version` / `cat /opt/mellanox/doca/version` | SDK $\neq$ Driver/Firmware version |
| **L3: Build** | Compiler/Headers | `pkg-config --cflags doca-common` | Header not found / syntax error |
| **L4: Link** | Symbol Resolution | `ld` / `nm -u` | `undefined reference to doca_*` |
| **L5: Runtime** | OS/Env State | `doca_caps` / `hugepages` check | `DOCA_ERROR_BAD_STATE` / No hugepages |
| **L6: Program** | Logic/API Use | `doca_error_get_descr()` | `DOCA_ERROR_INVALID_PARAM` |
| **L7: Driver** | HW/Firmware | `mlxconfig` / `ibstat` | Port DOWN / Firmware mismatch |

## 2. Observability & Verbosity
Do not guess why a program is silent. Use the official verbosity surface.

- **SDK-level logs**: Use `--sdk-log-level <level>` when running tools.
- **Library-level traces**: Use the `doca-<lib>-trace` build flavor (if available) for deep internal packet/event logs.
- **Runtime Environment**: Set `DOCA_LOG_LEVEL` environment variable.

## 3. Debugging inside NGC Containers
If the user is in `nvcr.io/nvidia/doca/doca`, apply these constraints:
- **Observable**: Build logs, API return codes, `doca_caps` output.
- **Hidden**: Host `/etc/` configs, direct hardware access (unless mapped), physical port states.
- **Critical Check**: If `hugepages` is empty inside the container, verify the host mount first.

## 4. Capture & Report Workflow
Before routing to the **NVIDIA DOCA Developer Forum**, ensure the following "Repro Bundle" is captured:
1. `doca-version` output.
2. Full build command and the exact `ld` / `gcc` error.
3. The specific `DOCA_ERROR_*` code and the string from `doca_error_get_descr()`.
4. The `doca_caps` snapshot of the target device.

## Related Skills
- [`doca-setup ## debug`](../doca-setup.md) $\rightarrow$ L1-L3 (Env-class)
- [`doca-programming-guide ## debug`](../doca-programming-guide.md) $\rightarrow$ L4-L6 (Program-class)
- [Library-specific skills] $\rightarrow$ Internal API state (e.g., Flow pipe trace)
