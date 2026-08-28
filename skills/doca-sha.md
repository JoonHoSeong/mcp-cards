---
name: doca-sha
description: Offload SHA-1, SHA-256, and SHA-512 hashing to BlueField DPU/ConnectX hardware for high-throughput data integrity verification.
license: Apache-2.0
metadata:
  kind: library
  layer: feature
  dependencies:
    - doca-common
  routes_to:
    - doca-programming-guide
    - doca-debug
compatibility: >
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-sha`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA SHA (Secure Hash Algorithm)

`doca-sha` offloads the computation of SHA-1, SHA-256, and SHA-512 digests to the hardware accelerator, allowing the CPU to handle other tasks while the DPU processes multi-GiB data streams.

## 1. The SHA Model: One-Shot vs. Partial
Depending on the size of the input data and the device capabilities, you must choose between two execution modes.

### 1.1. One-Shot Hash (`doca_sha_task_hash`)
Used when the entire input buffer fits within the device's maximum supported source buffer size.
- **Workflow**: Submit one task $\rightarrow$ Receive one digest.
- **Constraint**: Bound by `doca_sha_cap_get_max_src_buf_size`.

### 1.2. Partial / Incremental Hash (`doca_sha_task_partial_hash`)
Used for streaming data or inputs that exceed the one-shot buffer limit.
- **Workflow**: Submit multiple partial tasks $\rightarrow$ Finalize with a completion task $\rightarrow$ Receive one digest.
- **Constraint**: Requires managing the internal state between partial submissions.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Common Setup**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **Algorithm Selection**: Choose between `SHA1`, `SHA256`, or `SHA512`.
3. **Capability Check**: 
    - Call `doca_sha_cap_task_hash_get_supported(devinfo, alg)` for one-shot.
    - Call `doca_sha_cap_task_partial_hash_get_supported(devinfo, alg)` for partial.
4. **Sizing**:
    - Query `doca_sha_cap_get_max_src_buf_size` to decide between One-Shot and Partial.
    - Query `doca_sha_cap_get_min_dst_buf_size` to allocate the destination digest buffer.

### 2.2. Execution (`run`)
1. **Buffer Prep**: Register `doca_buf` for source and destination.
2. **Permissions**:
    - Source: `DOCA_ACCESS_FLAG_LOCAL_READ_ONLY`.
    - Destination: `DOCA_ACCESS_FLAG_LOCAL_READ_WRITE`.
3. **Submit**: Submit the selected task type to the `doca_pe`.
4. **Drain**: Call `doca_pe_progress()` until the digest completion event is received.

## 3. Safety & Error Taxonomy

### 3.1. Common Error Patterns
- **`DOCA_ERROR_INVALID_VALUE`**: Typically caused by:
    - Using an unsupported algorithm on the current hardware.
    - Source buffer exceeding `max_src_buf_size` in one-shot mode.
    - Destination buffer smaller than `min_dst_buf_size` for the chosen algorithm.
- **`DOCA_ERROR_NOT_PERMITTED`**: Memory permission mismatch on the `doca_buf`.
- **`BAD_STATE` (Partial Hash)**: Occurs if partial tasks are submitted out of order or if the finalization task is called without prior partials.

### 3.2. Validation Strategy
Always validate the hardware digest against a known-good CPU-generated fixture (e.g., using `sha256sum` or NIST test vectors) before deploying to production.

## 4. Path Selection: When NOT to use doca-sha
- **Non-SHA Algorithms**: For SM3, BLAKE2, or SHA-3 $\rightarrow$ use a CPU library (OpenSSL).
- **Small Inputs**: For inputs under a few KiB, the DMA setup overhead outweighs the hardware acceleration benefit.
- **Encryption**: If you need both hashing and encryption $\rightarrow$ use [`doca-aes-gcm`](doca-aes-gcm.md).

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md) for hardware-level hashing hangs.
