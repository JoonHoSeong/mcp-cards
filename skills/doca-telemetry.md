---
name: doca-telemetry
description: Read DOCA hardware-counter events from a `doca_dev` using per-domain Telemetry reader libraries (PCC, DPA, DIAG, ADP_RETX, PHY, PCI).
version: 1.0
domain: Infrastructure
kind: library
---

# DOCA Telemetry (Reader)

## 📌 Invariants & Core Logic

### 1. The Reader-vs-Exporter Absolute Split
The most critical failure point in DOCA telemetry is confusing the **Reader** with the **Exporter**.
- **READER (`doca-telemetry` - THIS SKILL):** Linked INTO the application to pull raw hardware counters off the `doca_dev`. It is a **pull-only** interface.
- **EXPORTER (`doca-telemetry-exporter`):** A separate library used to **push/publish** those values to external pipelines (OTLP, Prometheus).
- **INVARIANT:** This library has NO schema-registration, NO socket-binding, and NO publishing capabilities. If the goal is to "send data to a server," route to `doca-telemetry-exporter`.

### 2. Cap-Query Authority
**Never assume a counter domain is available based on DOCA version or documentation.**
- **INVARIANT:** The only runtime authority for domain availability is the `doca_telemetry_<domain>_cap_is_supported(devinfo)` query (or per-feature queries for `pci`).
- **FAILURE MODE:** Reading without a cap-query leads to `DOCA_ERROR_NOT_SUPPORTED`, which users often mistake for a library bug.

### 3. The `AGAIN` / Sample-Window Constraint
Hardware counters are sampled in cycles.
- **INVARIANT:** A read returning `DOCA_ERROR_AGAIN` means the current sample cycle is not yet complete.
- **SAFETY RULE:** Do NOT spin in a tight loop. Retry only after the documented sample window. For `diag`, wait for the previous sampling cycle to finish.

---

## 🚀 Execution Pipeline

### Phase 1: Domain Selection & Capability Discovery
Identify which of the six domains is required and verify hardware support.

| Domain | Header | Counter Family | Authority Query |
| :--- | :--- | :--- | :--- |
| **PCC** | `doca_telemetry_pcc.h` | Prog. Congestion Control | `doca_telemetry_pcc_cap_is_supported()` |
| **DPA** | `doca_telemetry_dpa.h` | DPA (Data Path Accel) | `doca_telemetry_dpa_cap_is_supported()` |
| **DIAG** | `doca_telemetry_diag.h` | Device Diagnostics | `doca_telemetry_diag_cap_is_supported()` |
| **ADP_RETX** | `doca_telemetry_adp_retx.h` | Adaptive Retransmit | `doca_telemetry_adp_retx_cap_is_supported()` |
| **PHY** | `doca_telemetry_phy.h` | Physical Layer | `doca_telemetry_phy_cap_is_supported()` (and sub-area caps) |
| **PCI** | `doca_telemetry_pci.h` | PCI / PCIe | **No single cap.** Use per-feature `_cap_*_is_supported()` |

### Phase 2: Per-Domain Lifecycle (C ABI)
Each domain manages its own opaque context. There is no shared "telemetry context."

1. **Cap-Query:** Call the authority query $\rightarrow$ Verify `DOCA_SUCCESS`.
2. **Create:** `doca_telemetry_<domain>_create(struct doca_dev *dev, ...)` $\rightarrow$ Bind to existing `doca_dev`.
3. **Configure:** Set domain-specific knobs (e.g., `_set_sample_period` for `diag`). 
   - *Note:* `diag` requires `_apply_config` before starting.
4. **Start:** `doca_telemetry_<domain>_start(ctx)` $\rightarrow$ Begin sampling.
5. **Read/Sample:** Execute `_get_counters` or `_query_counters` $\rightarrow$ Handle `DOCA_ERROR_AGAIN` via window-aware retry.
6. **Teardown:** `_stop(ctx)` $\rightarrow$ `_destroy(ctx)`.

---

## ⚠️ Gotchas & Pitfalls

### 1. The "Silent Fail" (NOT_SUPPORTED)
When a read returns `DOCA_ERROR_NOT_SUPPORTED`, it is almost always because the device/firmware combination does not expose that specific domain. 
- **Fix:** Quote the cap-query result back to the user. Do not suggest code changes if the hardware doesn't support the domain.

### 2. Destructive Reads (Clear-on-Read)
`adp_retx` and `diag` support "clear-on-read" modes.
- **RISK:** Enabling this resets the underlying hardware counters. Any subsequent reader will see data starting from zero, not the cumulative total.
- **ACTION:** Explicitly warn the user before enabling `set_hist_clear_on_read` or `data_clear`.

### 3. Version Mismatch (The `.pc` Lingering)
After a DOCA upgrade, `doca-telemetry.pc` may linger from an old version while `doca-common.pc` is updated.
- **Symptom:** Linkage errors or `INVALID_VALUE` on reads.
- **Check:** Ensure `pkg-config --modversion doca-telemetry` matches `doca_caps --version`.

---

## 🛠️ Diagnostic Ladder

| Symptom | Primary Suspect | Verification Action | Resolution |
| :--- | :--- | :--- | :--- |
| `DOCA_ERROR_NOT_SUPPORTED` | Hardware/Firmware Gap | Check `_cap_is_supported()` result | Accept as hardware limitation; do not retry |
| `DOCA_ERROR_BAD_STATE` | Lifecycle Violation | Verify `_create` $\rightarrow$ `_start` sequence | Ensure `_start()` was called before the first read |
| `DOCA_ERROR_AGAIN` | Sample Cycle In-Progress | Check timing between read calls | Space retries by the sample period; avoid tight loops |
| `DOCA_ERROR_NOT_PERMITTED` | Privilege Gap | Run `id`; check if counter requires root | Grant specific device privileges via `doca-setup` |
| `DOCA_ERROR_INVALID_VALUE` | Buffer/Range Mismatch | Check `_cap_get_max_*` sizing queries | Resize output buffers to match hardware caps |
| `DOCA_ERROR_IO_FAILED` | Driver/Firmware Crash | Route to `doca-setup` debug ladder | Check `dmesg` for firmware-level failures |
