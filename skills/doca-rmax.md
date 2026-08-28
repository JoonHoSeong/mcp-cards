---
name: doca-rmax
description: Integrate and manage timing-precise media-over-IP streams (SMPTE ST 2110, market data) using the DOCA Rivermax integration on BlueField DPUs.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField, Rivermax, SMPTE ST 2110, Media-over-IP, Timing-Precise, RoCE]
---

# Skill Card: doca-rmax

## 1. Invariants & Core Model

### DOCA Rivermax Definition
DOCA Rivermax is an integration surface that allows DOCA applications to leverage the NVIDIA Rivermax SDK for **timing-precise media-over-IP streaming** (e.g., SMPTE ST 2110 video/audio, high-frequency market data). 
- **Core Role:** It provides a DOCA-native wrapper around the Rivermax SDK to enable sub-microsecond jitter and precise packet placement.
- **Fundamental Constraint:** `doca-rmax` is **receive-only** from the DOCA perspective. There is no transmit/output stream object in the public DOCA Rivermax API.

### The Hard Dependency: Rivermax SDK & License
Unlike most DOCA libraries, `doca-rmax` does **not** bundle the Rivermax functionality. It is a wrapper.
- **Invariant:** The NVIDIA Rivermax SDK must be separately installed, and a valid Rivermax license must be present and readable on the host.
- **Failure Mode:** Without the SDK and license, `doca_rmax_init()` will fail immediately, regardless of the correctness of the DOCA-side code.

### Integration Topology (The Three-Layer Model)
To get packets into a Rivermax stream, three distinct DOCA surfaces must be aligned:
1. **Steering Layer (`doca-flow`):** Directs incoming packets from the wire to the correct queue.
2. **Queue Layer (`doca-eth`):** The underlying transport queue that carries the packets.
3. **Integration Layer (`doca-rmax`):** The Rivermax session that processes the packets for timing and synchronization.

---

## 2. Execution Pipeline

### Phase 1: Precondition Verification (`configure`)
1. **SDK & License Audit:** Verify the Rivermax SDK is installed at the expected location and the license file is valid.
2. **Capability Query:** Call `doca_rmax_get_*_supported()` (e.g., for PTP clock or hardware packet-placement) to confirm the hardware and SDK version support the required timing features.
3. **Env Prep:** Ensure the application has necessary privileges (e.g., `sudo` or `mlnx` group) to open the `doca_dev`.

### Phase 2: Session Lifecycle (`run`)
1. **Global Init:** Initialize the Rivermax engine using `doca_rmax_init()`.
2. **Stream Creation:** Create a receive-only session using `doca_rmax_in_stream_create()`.
3. **Context Conversion:** Convert the stream to a DOCA context via `doca_rmax_in_stream_as_ctx()` before calling `doca_ctx_start()`.
4. **Property Tuning:** Set stream properties (e.g., timing offsets, stream IDs) using `doca_rmax_in_stream_set_*`.

### Phase 3: Data Path Alignment (`integrate`)
1. **Flow Rule Setup:** Program `doca-flow` rules to steer the target media stream to the queue associated with the Rivermax session.
2. **Queue Configuration:** Configure the `doca-eth` queue to handle the expected packet rate and size.
3. **Traffic Verification:** Monitor the Rivermax stream for incoming frames/events.

### Phase 4: Teardown & Release (`rollback`)
1. **Context Stop:** Call `doca_ctx_stop()` on the associated context.
2. **Stream Destroy:** Destroy the `doca_rmax_in_stream` session.
3. **Global Release:** Call `doca_rmax_release()` to shut down the Rivermax engine.

---

## 3. Gotchas & Critical Constraints

### High-Risk Anti-Patterns
- **Assuming Bundled SDK:** Attempting to use `doca-rmax` without a separate Rivermax SDK installation. This is the most common point of failure.
- **Ignoring Steering:** Setting up the Rivermax stream but forgetting to program the `doca-flow` rules. The result is a "clean" stream object that never receives packets.
- **Best-Effort Fallback:** Attempting to use `doca-eth` alone for timing-precise streams. This will lead to jitter that exceeds SMPTE ST 2110 requirements.

### Performance & Stability
- **Scheduling Discipline:** The underlying Rivermax stack requires **real-time priority** for streaming threads to maintain sub-microsecond jitter.
- **Versioning Axis:** Capability support is the intersection of the **DOCA version** AND the **Rivermax SDK version**. Always use `doca_rmax_get_*_supported()` rather than relying on version numbers.

---

## 4. Diagnostic Ladder

### Layer 1: Initialization Failures
- **Symptom:** `doca_rmax_init()` returns `DOCA_ERROR_NOT_SUPPORTED` or a license error.
- **Check:** Is the Rivermax SDK installed? Is the license file present and valid?
- **Fix:** Install Rivermax SDK and apply the valid license.

### Layer 2: Stream Setup Errors
- **Symptom:** `doca_rmax_in_stream_create()` fails.
- **Check:** Is the `doca_dev` opened against a supported port/representor?
- **Fix:** Verify port capabilities and device handle.

### Layer 3: No Traffic / Silent Stream
- **Symptom:** Stream is active, but no packets are received.
- **Check:** Are there active `doca-flow` rules steering traffic to this queue? Is the `doca-eth` queue configured correctly?
- **Fix:** Program the steering rules via `doca-flow`.

### Layer 4: Timing & Jitter Issues
- **Symptom:** Packets arrive, but with excessive jitter or synchronization loss.
- **Check:** Is the thread running with real-time priority? Is the PTP clock configured correctly?
- **Fix:** Adjust thread scheduling and verify PTP synchronization.

---

## 5. Command Reference

| Call / Command | Purpose | Location |
| :--- | :--- | :--- |
| `doca_rmax_init()` | Initializes the global Rivermax engine. | Host API |
| `doca_rmax_in_stream_create()` | Creates a receive-only input stream session. | Host API |
| `doca_rmax_in_stream_as_ctx()` | Converts the stream to a DOCA Core context. | Host API |
| `doca_rmax_get_*_supported()` | Queries hardware/SDK support for specific timing features. | Host API |
| `pkg-config doca-rmax` | Resolves library include/link flags. | Host Shell |

## 6. Related Skills
- [`doca-eth`](../doca-eth/SKILL.md): The queue surface that carries Rivermax packets.
- [`doca-flow`](../doca-flow/SKILL.md): The steering surface that directs packets to the Rivermax queue.
- [`doca-setup`](../../doca-setup/SKILL.md): General DOCA installation and env prep.
- [`doca-debug`](../../doca-debug/SKILL.md): Cross-cutting debug ladder for DOCA.
- [`doca-version`](../../doca-version/SKILL.md): Canonical version-handling rules.
EOF
