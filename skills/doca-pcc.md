---
name: doca-pcc
description: Load, parameterize, and manage custom Programmable Congestion Control (PCC) algorithms on BlueField DPU ports using the host-side doca-pcc library.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField, PCC, DPA, RoCE, Congestion Control]
---

# Skill Card: doca-pcc

## 1. Invariants & Core Model

### The Two-Side-Program Model (Non-Negotiable)
Every custom PCC deployment consists of two distinct translation units that must be synchronized:
- **Host Side (`doca-pcc` library):** Written in C/C++, handles the lifecycle: creation of `doca_pcc` context $\rightarrow$ loading algorithm image $\rightarrow$ parameterization $\rightarrow$ starting the context.
- **DPA Side (Custom Algorithm):** Written in DPA-specific source, compiled by `dpacc` into a binary image (`doca_pcc_app`). This code executes per-packet/per-event on the BlueField DPA processor.

**Agent Rule:** Never propose computing rate-updates on the host side. The host *configures*; the DPA *executes*.

### Triple-Axis Capability Requirement
A custom PCC deployment is only possible if ALL THREE axes are satisfied:
1. **Hardware Axis:** BlueField generation must have a DPA processor exposed to the host.
2. **Firmware Axis:** The BlueField firmware **must** have the `custom-PCC slot` enabled. (Common failure point: `DOCA_ERROR_NOT_PERMITTED`).
3. **Software Axis:** DOCA install and `dpacc` compiler versions must match per the DOCA Compatibility Policy.

### Lifecycle Invariant
The `doca_pcc` object is **per-port**. There is no global PCC context.
`doca_pcc_create` $\rightarrow$ `doca_pcc_app` load $\rightarrow$ Parameterize $\rightarrow$ `doca_pcc_start` $\rightarrow$ `doca_pcc_stop` $\rightarrow$ `doca_pcc_destroy`.

---

## 2. Execution Pipeline

### Phase 1: Environment & Capability Audit (`configure`)
1. **Verify Versions:** Check `pkg-config --modversion doca-pcc` and `dpacc --version`. Cross-reference with the DOCA Compatibility Policy.
2. **Hardware Probe:** Run `doca_caps --list-devs` to ensure the target BlueField port exposes a DPA.
3. **Firmware Check:** Explicitly verify the `custom-PCC slot` is enabled. If disabled, route to `doca-setup` for firmware reconfiguration and required BlueField reset.
4. **API Query:** Execute `doca_pcc_cap_*` against the active `doca_devinfo` to confirm runtime support.

### Phase 2: Build & Integration (`build`)
1. **DPA Compilation:** Use `dpacc` to compile the DPA-side algorithm source into the binary image.
2. **Host Linking:** Link the host executable against `doca-pcc` using `pkg-config --libs doca-pcc`.
3. **Binary Embedding:** Ensure the `dpacc` output is correctly embedded as the `doca_pcc_app` within the host binary.
4. **Avoid Partial Rebuilds:** If either the DPA source or DOCA version changes, **rebuild both sides**. Partial rebuilds cause `DOCA_ERROR_DRIVER` or silent on-wire misbehavior.

### Phase 3: Deployment & Activation (`run`)
1. **Context Creation:** Call `doca_pcc_create` against the specific `doca_dev` mapping to the traffic-bearing port.
2. **Image Loading:** Load the `doca_pcc_app` into the context.
3. **Parameterization:** Set algorithm-specific knobs via the host API.
4. **Activation:** Call `doca_pcc_start()`. This is the exact moment the algorithm binds to live RDMA/RoCE traffic.
5. **PE Progression:** Ensure the host-side `doca_pe_progress` loop is running to drain algorithm reports.

### Phase 4: Verification & Tuning (`test`)
1. **Smoke Test (Trivial):** Deploy a no-op or pass-through algorithm. Verify load/start success and receipt of at least one host report.
2. **Traffic Smoke:** Introduce RDMA/RoCE traffic. Use the `pcc_counters` CLI to verify the algorithm is actually modulating traffic.
3. **Negative Tests:**
    - Pass out-of-range parameters $\rightarrow$ Expect `DOCA_ERROR_INVALID_VALUE`.
    - Query unsupported capabilities $\rightarrow$ Expect `DOCA_ERROR_NOT_SUPPORTED`.

---

## 3. Gotchas & Critical Constraints

### Common Failure Modes
- **`DOCA_ERROR_NOT_PERMITTED`**: Almost always means the **firmware custom-PCC slot is disabled**, NOT a Linux permission issue.
- **`DOCA_ERROR_DRIVER`**: Typically caused by **DOCA vs. DPACC version skew** or a mismatched algorithm image.
- **"Loaded but No Effect"**: Usually caused by:
    - No actual RDMA/RoCE traffic on the attached port (route to `doca-rdma`).
    - `doca_pcc_start()` was never called.
    - The DPA-side algorithm body has no effect path.

### Safety & Hardware Policy
- **No Inventing Algorithms:** The agent must refuse to design congestion control logic. Route "what should I compute" questions to the public DOCA PCC guide.
- **Isolated Testing:** Only run traffic-affecting algorithms in approved, isolated test scopes.
- **Teardown Order:** Must release `doca_pcc_app` **BEFORE** destroying the `doca_pcc` context to avoid `DOCA_ERROR_BAD_STATE`.

---

## 4. Diagnostic Ladder

### Layer 1: Basic Env (Standard DOCA)
- **Check:** `pkg-config` and `doca_caps`.
- **Tool:** `doca-version`, `doca-setup`.

### Layer 2: PCC-Specific Hardware/Firmware
- **Symptom:** `DOCA_ERROR_NOT_PERMITTED` or `NOT_SUPPORTED`.
- **Check:** Firmware custom-PCC slot status $\rightarrow$ BlueField Generation $\rightarrow$ DPA exposure.
- **Fix:** `doca-setup` (firmware flip + reset).

### Layer 3: Build & Linkage
- **Symptom:** `DOCA_ERROR_DRIVER` or link-time symbols missing.
- **Check:** `doca-pcc` version vs. `dpacc` version.
- **Fix:** Rebuild both sides against matched versions.

### Layer 4: Runtime Lifecycle
- **Symptom:** `DOCA_ERROR_BAD_STATE` or no reports.
- **Check:** `doca_pcc_start()` called? $\rightarrow$ `doca_pe_progress` running? $\rightarrow$ Teardown order correct?

### Layer 5: On-Wire Effect
- **Symptom:** Host reports "OK" but traffic is unchanged.
- **Check:** `pcc_counters` CLI output.
- **Fix:** Verify RDMA/RoCE traffic presence via `doca-rdma`.

---

## 5. Command Reference

| Class | Command | Purpose |
| :--- | :--- | :--- |
| **Version** | `pkg-config --modversion doca-pcc` | Build-time library version. |
| **Build** | `pkg-config --cflags --libs doca-pcc` | Host link/include flags. |
| **Toolchain** | `dpacc --version` | Compiler version (must match DOCA). |
| **Hardware** | `doca_caps --list-devs` | Identify DPA-capable BlueField ports. |
| **Debug** | `DOCA_LOG_LEVEL=trace ./binary` | Trace host-side lifecycle transitions. |
| **Observability**| `pcc_counters` (via `doca-public-knowledge-map`) | Read-only port counters (Infrastructure-side). |
| **DPA Debug** | DPA Debugger/Inspector (via `doca-dpa`) | Inspect stuck DPA kernels. |

## 6. Related Skills
- [`doca-setup`](../../doca-setup/SKILL.md): Firmware slot enable, DPA mode, install.
- [`doca-dpa`](../doca-dpa/SKILL.md): Generic DPA lifecycle and DPACC compiler.
- [`doca-rdma`](../doca-rdma/SKILL.md): Standing up the traffic that PCC modulates.
- [`doca-version`](../../doc
