# 🎴 Skill Card: doca-version (Ultra-Detailed)

**Use this skill when diagnosing version mismatches, verifying compatibility between the host and DPU, or interpreting NGC container tags.** This skill is the primary tool for resolving "silent failures"—where a binary compiles and runs but fails to execute hardware tasks due to a version drift between the SDK, the installed packages, and the DPU firmware.

Trigger this skill for requests like *"why am I getting DOCA_ERROR_NOT_SUPPORTED"*, *"check if my DPU firmware is compatible with DOCA 3.3"*, *"explain the NGC container tag"*, or *"verify if my build and runtime versions match"*.

---

## 1. Core Architecture: The 4-Way Match

A healthy DOCA environment requires strict synchronization across four distinct layers. If any one of these is out of sync, the system is in a state of **Version Drift**, which often leads to undefined behavior or `DOCA_ERROR_NOT_SUPPORTED`.

### A. The Canonical Detection Chain
You must verify all four sources. Do not summarize or omit any of these steps.

| Layer | Source | Command / File | Meaning |
| :--- | :--- | :--- | :--- |
| **1. Build-side** | `pkg-config` | `pkg-config --modversion doca-common` | The version the compiler used to link the binary. |
| **2. Install-side** | VERSION File | `cat /opt/mellanox/doca/applications/VERSION` | The version of the libraries currently on disk. |
| **3. Tool-side** | Capabilities Tool | `doca_caps --version` | The version of the SDK utilities used for probing. |
| **4. Hardware-side** | BlueField OS/FW | `bfver` OR `cat /etc/mlnx-release` | The actual firmware/OS version running on the DPU. |

**Consistency Rule:** All four sources must share the same Major and Minor version. If they differ, the environment is "Inconsistent."

### B. NGC Container Semantics
When using the NVIDIA GPU Cloud (NGC) containers:
- **Tag Warning:** Avoid the `latest` tag. It is mutable and leads to non-reproducible builds. Always use specific version tags (e.g., `3.3.0`).
- **Internal Alignment:** Inside the container, Layers 1, 2, and 3 are pre-aligned. However, Layer 4 (the physical DPU) is external. **The most common failure is a mismatch between the Container Tag and the physical BFB version.**

---

## 2. Implementation Paths: Drift Diagnosis

### Path 1: The "Build-vs-Runtime" Audit
Use this path when a binary compiles successfully but fails during execution.
1. **Build Check:** Run `pkg-config --modversion doca-common` on the build machine.
2. **Runtime Check:** Run `cat /opt/mellanox/doca/applications/VERSION` on the target DPU.
3. **Comparison:** If the Build version > Runtime version, the binary may be calling symbols that do not exist in the runtime libraries (`undefined reference` or `segmentation fault`).

### Path 2: The "SDK-vs-Hardware" Audit
Use this path when `doca_caps` reports a feature is unsupported, but the documentation says it should be.
1. **Tool Check:** Run `doca_caps --version`.
2. **HW Check:** Run `bfver`.
3. **Comparison:** If the SDK is newer than the BFB, the tool may be attempting to probe hardware features that do not exist in the older firmware.

---

## 3. Pre-flight Checklist

- [ ] **4-Way Audit Completed:** Have all four detection commands been executed?
- [ ] **Tag Specificity:** If using a container, is the version tag explicitly recorded (not `latest`)?
- [ ] **BFB Baseline:** Is the current BFB version documented before attempting an update?
- [ ] **Compatibility Ref:** Is the official DOCA Compatibility Policy URL open for reference?

---

## 4. Critical Rules & Troubleshooting

### A. Absolute Operational Rules
1. **No Version Guessing:** Never assume the version based on a date or a general "recent install." Always request the output of the 4-way detection chain.
2. **Anti-Hallucination:** Do not suggest non-existent version-check commands (e.g., `bfb-info` or `mlxprivhost`). Stick strictly to `bfver` and `mlnx-release`.
3. **Conservative Updates:** Never recommend a blind `apt upgrade`. Updating the host SDK without matching the BFB firmware can break the entire environment.

### B. Version Drift Diagnosis Matrix

| Symptom | Probable Root Cause | Resolution Path |
| :--- | :--- | :--- |
| `DOCA_ERROR_NOT_SUPPORTED` | BFB version is too old for the SDK version. | Update the BFB bundle to match the SDK version. |
| `undefined reference` at runtime | Build version $\neq$ Install version. | Re-compile the binary using the libraries present on the target DPU. |
| No packets moving (Silent Fail) | Version mismatch in the data-plane drivers. | Sync the host-side drivers with the DPU's firmware version. |
| `pkg-config` vs `apt` mismatch | Shell session not refreshed or duplicate paths. | Run `hash -r` and audit `PKG_CONFIG_PATH`. |

### C. Debugging Ladder
1. **L1 (Consistency):** Do the 4-Way Match values align?
2. **L2 (Symbolism):** If there is a crash, does `nm -D <binary>` show symbols that the runtime library lacks?
3. **L3 (Capability):** Does `doca_caps` see the hardware, or is it failing due to a driver mismatch?
4. **L4 (Policy):** Does the official Compatibility Matrix permit this specific version pairing?

---

## 5. Final Verification Steps

- [ ] **Consistency Check:** Do all 4 layers now report the same version?
- [ ] **Functionality Check:** Does a basic "Hello World" sample now run without `DOCA_ERROR_NOT_SUPPORTED`?
- [ ] **Tag Lock:** Has the `latest` tag been replaced by a specific version tag in the deployment YAML?

---

## 6. Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `pkg-config --modversion doca-common` | Check Build-side version | `3.x.x` |
| `cat /opt/mellanox/doca/applications/VERSION` | Check Install-side version | `3.x.x` |
| `doca_caps --version` | Check Tool-side version | `3.x.x` |
| `bfver` | Check Hardware-side version | `3.x.x` |
