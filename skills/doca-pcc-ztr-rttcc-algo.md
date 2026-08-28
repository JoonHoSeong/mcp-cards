---
name: doca-pcc-ztr-rttcc-algo
description: Deploy, tune, and evaluate the NVIDIA-shipped Zero-Touch RoCE RTT-based Congestion Control (ZTR RTTCC) reference algorithm on BlueField-3 DPUs.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField-3, ZTR-RTTCC, PCC, RoCE-v2, DPA]
---

# Skill Card: doca-pcc-ztr-rttcc-algo

## 1. Invariants & Core Model

### The Reference Algorithm Baseline
ZTR RTTCC is the "no-config-required" baseline reference algorithm shipped by NVIDIA. It is a **library** consumed by the `doca-pcc` host-side framework.
- **Role:** It provides a pre-implemented congestion control logic that runs on the BlueField-3 DPA.
- **Constraint:** This skill is for deploying and tuning the *shipped* algorithm. Writing a custom algorithm from scratch is handled by `doca-pcc`.

### The Three-Artifact Triangle
A ZTR RTTCC deployment depends on three distinct surfaces. Conflating these is a primary failure mode:
1. **`doca-pcc-ztr-rttcc-algo` (This Skill):** The specific algorithm logic (DPA-side library).
2. **`doca-pcc` (Framework):** The host-side library that loads *any* algorithm image onto the DPA.
3. **`doca-pcc-counters` (Tool):** The read-only CLI for inspecting per-port/per-flow PCC counters.

### Hardware & Firmware Invariant
- **Target:** Specifically designed for **BlueField-3** DPUs.
- **Prerequisite:** The BlueField firmware **must** have the `custom-PCC slot` enabled (handled via `doca-setup`).
- **Traffic:** The algorithm modulates **existing** RoCE-v2 traffic. If no traffic is flowing on the port, the algorithm is inert.

---

## 2. Execution Pipeline

### Phase 1: Baseline Decision (`decide`)
1. **Assess Need:** Determine if the shipped ZTR RTTCC baseline is sufficient or if a custom algorithm is required based on latency/fairness/convergence targets.
2. **Environment Audit:** Verify DOCA install, `dpacc` compiler match, and BlueField-3 DPA exposure.

### Phase 2: Integration & Build (`modify` & `build`)
1. **Identify Sample:** Start with the shipped DOCA PCC sample (`/opt/mellanox/doca/applications/pcc/`).
2. **Patch Callbacks:** Apply the following minimum-diff edits to the sample's DPA-side code:
    - **Init:** Call `doca_pcc_dev_ztr_rttcc_init(algo_idx)` inside the `doca_pcc_dev_user_init()` callback.
    - **Dispatch:** Route the `doca_pcc_dev_user_algo()` callback to the `doca_pcc_dev_ztr_rttcc_algo` symbol.
    - **Params:** Implement `doca_pcc_dev_user_set_algo_params()` using `doca_pcc_dev_set_ztr_rttcc_params`.
3. **Select Variant:** Choose the algorithm variant (Vanilla, Path-Migration, RX-Rate, Multipath, or Window-Probeless) at **DPACC compile time**.
4. **Full Rebuild:** Rebuild both the DPA-side image (via `dpacc`) and the host executable to ensure binary synchronization.

### Phase 3: Deployment & Tuning (`run` & `use`)
1. **Load & Start:** Use the `doca-pcc` lifecycle to load the ZTR RTTCC image and start the context on the target port.
2. **Host-Side Tuning:** Adjust algorithm parameters (e.g., `BW_G`, `ALPHA`, `MAX_DEC`) using the host-side `doca_pcc_dev_set_ztr_rttcc_params` call without needing to rebuild the DPA image.
3. **Verify Modulation:** Use the `pcc_counters` CLI to confirm the algorithm is actively shaping RoCE-v2 flows.

### Phase 4: Evaluation & Debugging (`test` & `debug`)
1. **Smoke Test:** Deploy the vanilla variant and confirm it reports status via the host PE.
2. **Traffic Validation:** Introduce sustained RoCE-v2 load and verify the "Zero-Touch" convergence behavior.
3. **Failure Analysis:** If `DOCA_PCC_DEV_STATUS_FAIL` occurs, check the DPA-side initialization sequence and parameter range validity.

---

## 3. Gotchas & Critical Constraints

### High-Risk Anti-Patterns
- **Algorithm Logic Migration:** Never attempt to move the RTT-calculation or rate-update logic from the DPA to the host side.
- **Partial Rebuilds:** Updating the DPA variant without rebuilding the host executable often leads to `DOCA_ERROR_DRIVER`.
- **Assuming "Loaded == Working":** A successful `doca_pcc_start()` only means the image is live. On-wire effect must be verified via `pcc_counters`.

### Parameter Constraints
- **Range Validation:** Parameters passed via `doca_pcc_dev_set_ztr_rttcc_params` are range-checked on the DPA. Invalid values will cause the call to return a failure status.
- **Variant Lock-in:** The choice between "Multipath" and "Vanilla" is fixed at compile time; you cannot switch variants via host-side parameters.

---

## 4. Diagnostic Ladder

### Layer 1: Artifact Identification
- **Symptom:** Confusing the algorithm with the framework or the tool.
- **Check:** Is the user trying to *load* (Framework), *tune* (Algorithm), or *inspect* (Tool)?
- **Fix:** Route to the correct skill: `doca-pcc` $\rightarrow$ `doca-pcc-ztr-rttcc-algo` $\rightarrow$ `doca-pcc-counters`.

### Layer 2: Deployment Failures
- **Symptom:** `DOCA_PCC_DEV_STATUS_FAIL` during initialization.
- **Check:** Is the BlueField-3 `custom-PCC slot` enabled in firmware?
- **Fix:** `doca-setup` $\rightarrow$ Firmware flip $\rightarrow$ Reset.

### Layer 3: Integration Bugs
- **Symptom:** The application starts, but the algorithm is not dispatching.
- **Check:** Are the `doca_pcc_dev_user_*` callbacks correctly patched to the ZTR RTTCC symbols?
- **Fix:** Re-verify the README's minimum-diff edits.

### Layer 4: Performance Issues
- **Symptom:** RoCE-v2 flows are not being throttled or are unstable.
- **Check:** Are the host-set parameters (`ALPHA`, `MAX_INC`, etc.) appropriate for the workload?
- **Fix:** Tune via `doca_pcc_dev_set_ztr_rttcc_params`.

---

## 5. Command Reference

| Call / Command | Purpose | Location |
| :--- | :--- | :--- |
| `doca_pcc_dev_ztr_rttcc_init` | Initializes algorithm state on DPA. | DPA-side API |
| `doca_pcc_dev_ztr_rttcc_algo` | The main per-event CC logic body. | DPA-side API |
| `doca_pcc_dev_set_ztr_rttcc_params` | Updates algorithm knobs from host. | DPA-side API |
| `pkg-config doca-pcc-ztr-rttcc-algo` | Resolves library include/link flags. | Host Shell |
| `pcc_counters` | Inspects real-time PCC port counters. | `/opt/mellanox/doca/tools/` |

## 6. Related Skills
- [`doca-pcc`](../doca-pcc/SKILL.md): The host-side framework required to load this algorithm.
- [`doca-pcc-counters`](../../tools/doca-pcc-counters/SKILL.md): The canonical tool for verifying the algorithm's on-wire effect.
- [`doca-setup`](../../doca-setup/SKILL.md): Used to enable the mandatory firmware custom-PCC slot.
- [`doca-dpa`](../doca-dpa/SKILL.md): DPA-side programming and `dpacc` compiler discipline.
EOF
