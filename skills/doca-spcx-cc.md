---
name: doca-spcx-cc
description: Load, parameterize, and evaluate custom Programmable Congestion Control (SPCX) algorithms on BlueField DPUs using the `doca_spcx_cc` CLI tool.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField, SPCX, PCC, Congestion Control, RoCE, RDMA, DPA]
---

# Skill Card: doca-spcx-cc

## 1. Invariants & Core Model

### SPCX Definition & Role
`doca_spcx_cc` is the operator-side CLI tool used to deploy and manage **SPCX-class Programmable Congestion Control (PCC)** algorithms. It serves as the bridge between a DPA-compiled algorithm image and the live RDMA/RoCE fabric.
- **Core Role:** Loading, parameterizing, and observing the effect of custom congestion control logic on a BlueField DPU's DPA processor.
- **Invariant:** The tool is a **host-side harness**. It does not define the algorithm logic itself (which is written in DPACC) but manages its lifecycle on the hardware.

### The Programmable-CC Decision Tree
A critical invariant is choosing the correct surface. Conflating these is the most common first-touch error:
- **Factory PCC:** Firmware-resident default. No host-side tool/library needed. Configured via firmware knobs.
- **`doca-pcc`:** The established, stable programmable surface.
- **`doca-spcx-cc` (SPCX):** The next-generation, more flexible extension. Used for newer, more complex algorithms that require SPCX-specific capabilities.

### The "No Signal Under No Contention" Rule
A fundamental physical invariant of congestion control: **A CC algorithm has no signal to act upon if there is no contention on the fabric.**
- **Invariant:** If the tool reports the algorithm is `Active` but throughput/latency curves are unchanged, it is often because the fabric is under-utilized (no congestion), not because the algorithm is broken.

---

## 2. Execution Pipeline

### Phase 1: Precondition & Surface Audit (`configure`)
1. **Surface Selection:** Apply the decision tree (Factory $\rightarrow$ PCC $\rightarrow$ SPCX) to ensure `doca_spcx_cc` is the correct tool for the target algorithm.
2. **Hardware/Firmware Audit:** Verify the BlueField DPU has the **custom-PCC slot enabled** in firmware.
3. **Version Alignment:** Ensure the `doca_spcx_cc` tool, `doca-pcc` library, DPACC compiler, and firmware are all version-matched.
4. **Role Assignment:** Define the role of the DPU in the fabric (e.g., RP - Reaction Point / NP - Notification Point).

### Phase 2: Deployment & Smoke Test (`run`)
1. **Algorithm Loading:** Use `doca_spcx_cc` to load the DPACC-compiled algorithm image onto the DPA.
2. **Parameterization:** Set algorithm-specific parameters (e.g., gain, thresholds) via the CLI.
3. **Session Activation:** Start the SPCX session and verify the `Active` status via `--status`.
4. **Smoke Test:** Verify that the DPA is successfully intercepting the RDMA flow without crashing the session.

### Phase 3: Contention-Positive Evaluation (`test`)
1. **Replica Setup:** Deploy the algorithm on a non-production replica pair of BlueFields.
2. **Contention Injection:** Introduce controlled contention (e.g., using a traffic generator) to create a "congestion-positive" environment.
3. **Metric Observation:** Observe the throughput/latency curves and per-flow trace formats provided by the tool.
4. **Iterative Tuning:** Adjust parameters and re-evaluate until the desired CC behavior is achieved.

### Phase 4: Production Roll-forward (`use`)
1. **Safety Gate:** Ensure the "Blast Radius" is bounded and an Out-of-Band (OOB) management path is available.
2. **Rollback Rehearsal:** Document and test the procedure to revert to the Factory PCC algorithm.
3. **Observability Guard:** Confirm that runtime metrics are being captured and can be monitored in real-time.
4. **Controlled Cutover:** Deploy to production in stages, monitoring for fabric stability.

---

## 3. Gotchas & Critical Constraints

### High-Risk Anti-Patterns
- **Production First Deployment:** Loading a custom CC algorithm directly into a production fabric. A wrong algorithm can "melt" the fabric (cause total collapse).
- **Ignoring the Contention Rule:** Blaming the algorithm for "doing nothing" when the fabric is uncongested.
- **Version Mismatch:** Using a tool version that is incompatible with the DPACC-compiled image, leading to `DOCA_PCC_PS_ERROR`.

### Stability & Safety
- **DPA-Side Crash:** A bug in the custom algorithm can cause the DPA to hang or crash, which may impact the networking performance of the BlueField.
- **Fabric-Wide Impact:** Congestion control is a distributed protocol; an unstable algorithm on one node can destabilize the entire RoCE fabric.

---

## 4. Diagnostic Ladder

### Layer 1: Tool & Environment Failures
- **Symptom:** `doca_spcx_cc` command not found or fails to execute.
- **Check:** Is DOCA installed? Is the tool present in `/opt/mellanox/doca/tools/`?
- **Fix:** Install DOCA and verify the tool path.

### Layer 2: Device & Firmware Failures
- **Symptom:** `DOCA_PCC_PS_ERROR` on start or "custom-PCC slot not enabled".
- **Check:** Is the custom-PCC slot enabled in the BlueField firmware?
- **Fix:** Enable the custom-PCC slot via firmware configuration.

### Layer 3: Loading & Binding Failures
- **Symptom:** Algorithm image fails to load or is rejected by the DPA.
- **Check:** Was the image compiled with a version-matched DPACC compiler?
- **Fix:** Recompile the algorithm using the matching DPACC version.

### Layer 4: Runtime "Silent" Failures
- **Symptom:** Status is `Active`, but no change in network behavior.
- **Check:** Is there actual contention on the fabric?
- **Fix:** Inject contention using a traffic generator to create a signal for the algorithm.

---

## 5. Command Reference

| Command / Flag | Purpose | Location |
| :--- | :--- | :--- |
| `doca_spcx_cc --status` | Checks the current status of the SPCX session. | Host CLI |
| `doca_spcx_cc <algo_path> <params>` | Loads and starts a specific SPCX algorithm. | Host CLI |
| `doca_spcx_cc --stop` | Stops the currently running SPCX algorithm. | Host CLI |
| `doca_spcx_cc --help` | Displays the authoritative list of flags and subcommands. | Host CLI |

## 6. Related Skills
- [`doca-pcc`](../../libs/doca-pcc/SKILL.md): The established programmable CC library; use for stable, non-SPCX algorithms.
- [`doca-pcc-ztr-rttcc-algo`](../../libs/doca-pcc-ztr-rttcc-algo/SKILL.md): A reference RTT-based algorithm that can be loaded via SPCX.
- [`doca-dpa`](../../libs/doca-dpa/SKILL.md): For DPA-side algorithm authoring and low-level DPA control.
- [`doca-rdma`](../../libs/doca-rdma/SKILL.md): The RDMA/RoCE surface being controlled by the SPCX algorithm.
- [`doca-hardware-safety`](../../doca-hardware-safety/SKILL.md): The meta-policy for deploying high-risk algorithms to production.
- [`doca-setup`](../../doca-setup/SKILL.md): For firmware configuration (custom-PCC slot) and tool installation.
EOF
