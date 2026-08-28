---
name: doca-sha-offload-engine
description: Offload one-shot SHA-1, SHA-256, and SHA-512 hashing from OpenSSL to DOCA SHA hardware using the OpenSSL ENGINE wrapper.
version: 1.0
domain: Infrastructure
category: DOCA
tags: [BlueField, OpenSSL, SHA, Offload, Engine, Cryptography]
---

# Skill Card: doca-sha-offload-engine

## 1. Invariants & Core Model

### The Offload Engine Concept
The `doca-sha-offload-engine` is a shared object (`libdoca_sha_offload_engine.so`) that implements the OpenSSL ENGINE API. It acts as a transparent proxy between OpenSSL's `EVP_Digest` interface and the underlying DOCA SHA hardware.
- **Key Value Proposition:** Allows existing OpenSSL-based applications to achieve hardware acceleration for SHA hashing **without rewriting the application source code**.
- **Invariant:** It is a **transparent wrapper**. If the engine is loaded and selected, OpenSSL routes the hash request to the hardware; otherwise, it falls back to the software implementation.

### Algorithm Coverage & Constraints
The engine is specifically designed for **one-shot hashing** via the `EVP_Digest` interface.
- **Supported Algorithms:** SHA-1, SHA-256, SHA-512.
- **Unsupported Algorithms:** SHA-224, MD5, SHA-3, HMAC-SHA.
- **Processing Model:** While it supports the `EVP_DigestUpdate` chain, it implements this by buffering the input and executing a **single one-shot hardware call** at `EVP_DigestFinal`. It is not a true streaming/incremental hardware offload.

### The PCIe Address Invariant
The engine must bind to a specific PCIe device where the DOCA SHA hardware resides.
- **Default:** `03:00.0` (as defined in the shipped source).
- **Requirement:** If the hardware is at a different address, the operator **must** either:
    1. Override `DOCA_ENGINE_PCI_ADDR` at build time.
    2. Use the `set_pci_addr` control command at runtime.

---

## 2. Execution Pipeline

### Phase 1: Environment & Prerequisite Audit (`configure`)
1. **OpenSSL Version Check:** Ensure OpenSSL $\ge$ 1.1.1 is installed. (Supports 1.1.1f and 3.0.2).
2. **Dependency Audit:** Verify `libssl-dev` (or equivalent) is present for build/link.
3. **Hardware Visibility:** Check that the DOCA SHA device is visible on the PCIe bus.

### Phase 2: Engine Deployment & Loading (`run`)
1. **Library Placement:** Ensure `libdoca_sha_offload_engine.so` is in the search path or specified explicitly.
2. **Engine Load (CLI):** Use `openssl engine dynamic -pre LOAD -id doca_sha` to register the engine.
3. **Engine Load (Programmatic):**
    - `ENGINE_load_dynamic()` $\rightarrow$ `ENGINE_by_id("doca_sha")` $\rightarrow$ `ENGINE_init()`.
4. **PCIe Address Binding:** If the device is not at `03:00.0`, call `ENGINE_ctrl_cmd_string` with the `set_pci_addr` command.

### Phase 3: Offload Verification (`test`)
1. **The Negative Test (The "Proof"):** Attempt to hash a message using **SHA-224**. 
    - Since the engine does NOT support SHA-224, OpenSSL will fall back to software.
    - If the engine is loaded and active, a specific failure or fallback behavior (compared to a non-engine run) proves the engine is intercepting calls.
2. **Performance Benchmarking:** Run `openssl speed sha256` with and without the `-engine doca_sha` flag.
3. **Message-Size Window Analysis:** Identify the threshold where hardware offload latency is lower than CPU processing time (hardware typically wins on larger blocks).

### Phase 4: Integration & Audit (`use`)
1. **Selection Rule:**
    - **Use Engine:** If you have an existing OpenSSL pipeline and want "drop-in" acceleration.
    - **Use `doca-sha` Library:** If you are building a new pipeline from scratch and need fine-grained control (e.g., partial hashes).

---

## 3. Gotchas & Critical Constraints

### High-Risk Anti-Patterns
- **Silent Software Fallback:** OpenSSL often fails silently back to software if the engine is improperly configured. Never assume offload is working just because the hash result is correct.
- **PCIe Address Mismatch:** Using the default `03:00.0` on a system where the device is located elsewhere leads to a `DOCA_ERROR` or a silent fallback.
- **Incremental Hashing Misconception:** Assuming the engine does real-time streaming offload for `EVP_DigestUpdate`. It buffers and hashes in one shot at the end.

### Stability & Compatibility
- **OpenSSL 3.x Deprecation:** While the ENGINE API is deprecated in OpenSSL 3.x, it is still supported via the legacy path. The engine is verified for OpenSSL 3.0.2.
- **Binary Compatibility:** Ensure the engine is built against the same OpenSSL headers used by the consuming application.

---

## 4. Diagnostic Ladder

### Layer 1: Loading Failures
- **Symptom:** `openssl engine` returns "Error loading engine".
- **Check:** Is the `.so` file path correct? Are the dependencies (`libssl`) resolvable?
- **Fix:** Check `ldd libdoca_sha_offload_engine.so` and verify the path in the `dynamic` load command.

### Layer 2: Binding Failures
- **Symptom:** Engine loads but `ENGINE_init()` fails or returns a PCIe-related error.
- **Check:** Does the PCIe address match the actual hardware location?
- **Fix:** Use the `set_pci_addr` control command to bind to the correct device.

### Layer 3: Performance Degradation
- **Symptom:** `openssl speed` is slower with the engine than without.
- **Check:** What is the message size? Small messages (< 4KB) often suffer from the PCIe round-trip overhead.
- **Fix:** Use the engine only for workloads with larger block sizes.

### Layer 4: Offload Verification
- **Symptom:** Uncertain if hardware is actually being used.
- **Check:** Use the `-engine_impl` flag in `openssl dgst` to force the implementation.
- **Fix:** Run the SHA-224 negative test to confirm the engine is the primary interceptor.

---

## 5. Command Reference

| Call / Command | Purpose | Location |
| :--- | :--- | :--- |
| `openssl engine dynamic -pre LOAD -id doca_sha` | Loads the DOCA SHA engine into the OpenSSL environment. | Host Shell |
| `openssl dgst -engine doca_sha -sha256 <file>` | Performs SHA-256 hashing using the DOCA SHA hardware. | Host Shell |
| `openssl speed -engine doca_sha sha256` | Benchmarks the hardware offload performance. | Host Shell |
| `ENGINE_ctrl_cmd_string(e, "set_pci_addr", "03:00.0")` | Binds the engine to a specific PCIe device. | Programmatic API |
| `pkg-config doca-sha` | Resolves the underlying `doca-sha` library flags. | Host Shell |

## 6. Related Skills
- [`doca-sha`](../../libs/doca-sha/SKILL.md): The underlying hardware library; use for new pipelines and fine-grained control.
- [`doca-version`](../../doca-version/SKILL.md): Canonical version-detection for DOCA and OpenSSL.
- [`doca-setup`](../../doca-setup/SKILL.md): OpenSSL and `libssl-dev` installation requirements.
- [`doca-debug`](../../doca-debug/SKILL.md): Cross-cutting debug ladder for DOCA.
- [`doca-hardware-safety`](../../doca-hardware-safety/SKILL.md): Hardware binding and PCIe safety policies.
EOF
