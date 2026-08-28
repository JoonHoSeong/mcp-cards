---
name: doca-structured-tools-contract
description: Define the detection, preference, and fallback behavior for using structured JSON tools vs. manual command chains across the DOCA ecosystem.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [Governance, JSON, Tooling, Detection, Fallback, DOCA-Env, Validation]
---

# Skill Card: doca-structured-tools-contract

## 1. Invariants & Core Model

### The "Prefer Structured" Philosophy
The `doca-structured-tools-contract` is a governance overlay. It ensures that the agent provides the most efficient and accurate answer by preferring a single, structured JSON output over a fragmented chain of manual shell commands.

### The Core Behavior Invariant: Detect $\rightarrow$ Prefer $\rightarrow$ Fallback $\rightarrow$ Report
Every agent interaction referencing this contract must follow this four-step loop:
1. **Detect:** Run a read-only probe (e.g., `command -v`) to see if the structured helper exists.
2. **Prefer:** If the probe succeeds AND the output matches the defined JSON schema, use the structured output as the sole source of truth.
3. **Fall back:** If the probe fails or the JSON is malformed/incomplete, execute the manual command chain defined in the schema section.
4. **Report:** Explicitly state which path was taken (e.g., *"Using structured `<tool>`"* or *"Falling back to manual chain..."*).

### Schema Immutability
- **Invariant:** Schemas are defined centrally in this skill.
- **Constraint:** Individual library/tool skills may *consume* these schemas in their Command appendices, but they **must not redefine** them. Any schema change must occur here first.

---

## 2. Execution Pipeline (Agent Workflow)

### Step 1: Probe & Detection
For the requested information (e.g., environment state), identify the matching schema and run the **Detection Probe**.
- *Example:* For environment state, run `command -v doca-env`.

### Step 2: Structured Path (The Fast Path)
If the probe succeeds:
1. Execute the structured tool (e.g., `doca-env --json`).
2. Validate the JSON against the schema (check required fields and types).
3. If valid, extract the data and skip the manual chain.
4. **Report:** *"Using structured `doca-env` (path: `/opt/mellanox/doca/bin/doca-env`)."*

### Step 3: Manual Fallback (The Reliable Path)
If the probe fails or validation fails:
1. Execute the **Manual Fallback Chain** in the exact order listed in the schema.
2. Synthesize the final answer by combining the results of each command.
3. If a critical command in the chain is missing, report the gap and route to `doca-setup`.
4. **Report:** *"Falling back to manual chain (structured `doca-env` probe failed: command not found)."*

### Step 4: Final Synthesis
Present the data to the user. If a manual chain was used, include a note suggesting the installation of the structured helpers for future efficiency.

---

## 3. Schemas & Fallback Chains

### A. Environment & Install State (`doca-env`)
- **Probe:** `command -v doca-env`
- **Structured Shape:** JSON object containing `version`, `devices` (PCIe address, kind, state), `libraries` (installed status, path), `drivers`, `hugepages`, `host_kind`, and `bf_mode`.
- **Manual Fallback Chain:**
    1. `pkg-config --modversion doca-common`
    2. `cat /opt/mellanox/doca/applications/VERSION`
    3. `doca_caps --version`
    4. `doca_caps --list-devs`
    5. `pkg-config` scan for all `.pc` files in the DOCA lib directory.
    6. `ls /opt/mellanox/doca/samples/`
    7. `lsmod | grep -E '^mlx5_(core|ib)'` & `uname -r`
    8. `cat /proc/meminfo | grep -i Huge`
    9. `dmidecode -s system-product-name` or `/proc/device-tree/model`
    10. `mlxconfig -d <pcie> q INTERNAL_CPU_MODEL`

### B. Capability Version Matrix (`version-matrix`)
- **Probe:** `test -f /opt/mellanox/doca/share/version-matrix.json`
- **Structured Shape:** JSON array of entries: `library`, `capability`, `display_name`, `min_doca_version`, `max_doca_version`.
- **Manual Fallback Chain:**
    1. Identify the library/capability via the per-library `CAPABILITIES.md`.
    2. Fetch the official doc page via `doca-public-knowledge-map`.
    3. Extract the "available since" prose.
    4. Compare against `pkg-config --modversion doca-<library>`.

### C. Per-Device Capability Snapshot (`capability-snapshot`)
- **Probe:** `command -v doca-capability-snapshot`
- **Structured Shape:** JSON object: `snapshot_at`, `doca_version`, `host_kind`, `devices` (map of library $\rightarrow$ capability flags).
- **Manual Fallback Chain:**
    1. `doca_caps --list-devs`
    2. For each device: execute the library-specific `doca_<lib>_cap_*` query via a small test program (following the library's `## test` workflow).

### D. Pre-Commit Validation (`validate-before-commit`)
- **Probe:** `command -v doca-validate`
- **Structured Shape:** JSON object: `library`, `spec_path`, `result` (pass/fail/skip), `checks` (name, status, details, remediation).
- **Manual Fallback Chain:**
    1. Locate the library-specific validation surface in the matching skill's `## test` workflow.
    2. If no read-only validator exists, report `result: skip` and route to the library's `## test` flow.

### E. Host vs. DPU State (`collect-state`)
- **Probe:** `command -v doca-collect-host-state` (on host) / `command -v doca-collect-dpu-state` (on DPU).
- **Structured Shape:** JSON object: `side`, `doca_version`, `firmware_version`, `kernel_version`, `mlx5_modules`, `bf_mode`, `devices`.
- **Manual Fallback Chain:**
    1. `doca_caps --version`
    2. `uname -r`
    3. `lsmod | grep mlx5`
    4. `devlink dev show` + `lspci` + `ip -j link`
    5. `flint -d <pcie> q` (privileged)
    6. `mlxconfig -d <pcie> q INTERNAL_CPU_MODEL`

---

## 4. Gotchas & Critical Constraints

### Privileges & Safety
- **Sudo is not Implicit:** Manual fallback commands requiring `sudo` (e.g., `flint`, `dmidecode`) must be executed via an approved privileged channel or requested from the user. Never silently elevate.
- **Read-Only Probes:** Probes must be strictly read-only. Never use a constructor (e.g., `_create`) as a probe.

### Data Integrity
- **No Field Invention:** The agent must not invent JSON fields. If the tool returns extra data not in the schema, treat it as advisory only.
- **Validation Failure:** If the structured tool exists but returns malformed JSON, the agent **must** report the validation failure and fall back to the manual chain.

---

## 5. Related Skills
- [`doca-setup`](../doca-setup/SKILL.md): The destination for users whose probes fail and who need to install the tools.
- [`doca-public-knowledge-map`](../doca-public-knowledge-map/SKILL.md): The source for manual capability version lookups.
- [`doca-version`](../../doca-version/SKILL.md): The canonical rules for version matching.
EOF
