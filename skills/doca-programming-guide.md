---
name: doca-programming-guide
description: Library-agnostic programming patterns for DOCA, including build systems, object lifecycles, and error handling.
license: Apache-2.0
metadata:
  kind: guide
  layer: foundation
  routing:
    first_app: modify-shipped-sample
    build: pkg-config-meson
---

# DOCA Programming Guide

This skill provides the **universal DNA** for all DOCA applications. Every library-specific skill (Flow, RDMA, etc.) inherits the patterns defined here.

## 1. The Canonical Build Pattern (C/C++)
Stop inventing build scripts. Use the `pkg-config` + `Meson` standard.

**The "Golden" Build Line:**
```bash
gcc my_app.c $(pkg-config --cflags --libs doca-common doca-flow) -o my_app
```
*Crucial: Always include `doca-common` first.*

## 2. The Universal Object Lifecycle
Every DOCA object (Pipe, Context, Buffer) follows this strict state machine. Deviating from this order causes `DOCA_ERROR_BAD_STATE`.

**Lifecycle: `Create` $\rightarrow$ `Init` $\rightarrow$ `Start` $\rightarrow$ `Use` $\rightarrow$ `Stop` $\rightarrow$ `Destroy`**

1. **Cfg-Create**: Allocate the descriptor/handle.
2. **Init**: Bind to hardware/resources.
3. **Start**: Activate the data path.
4. **Use**: Submit work / Poll for completion.
5. **Stop**: Quiesce the data path.
6. **Destroy**: Free memory/handles.

## 3. Error Taxonomy & Decoding
Never report a raw `DOCA_ERROR_*` integer. Always decode it.

**The Golden Rule:** Use `doca_error_get_descr(error_code)` immediately after any failed call.

- **Program-class Error**: `DOCA_ERROR_INVALID_PARAM` (User mistake).
- **Runtime-class Error**: `DOCA_ERROR_BAD_STATE` (Wrong lifecycle order).
- **Hardware-class Error**: `DOCA_ERROR_HW_FAILURE` (Driver/Firmware issue).

## 4. "First App" Derivation Workflow
Do not write a DOCA app from a blank file.
1. **Locate**: Find a shipped sample in `/opt/mellanox/doca/samples/<lib>/`.
2. **Copy**: Create a project directory and copy the `.c` and `meson.build` files.
3. **Prune**: Remove unnecessary features to reach the "Minimum Viable Application".
4. **Modify**: Implement the custom logic.

## 5. Safety Policy: Validate-Before-Commit
To prevent hardware crashes or "silent failures" (no packets on wire):
- Always call the `validate` API (if provided by the library) before calling `commit` or `start`.
- Check the return code of `start()`—a success here means the hardware is actually programmed.

## Related Skills
- [`doca-setup`](../doca-setup.md) $\rightarrow$ Env/Install verification.
- [`doca-debug`](../doca-debug.md) $\rightarrow$ Layered debug ladder.
- [Library Skills] $\rightarrow$ Specific API construction.
