---
name: doca-aes-gcm
description: Offload high-speed AES-GCM authenticated encryption and decryption to BlueField DPU/ConnectX hardware.
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
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-aes-gcm`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA AES-GCM (Authenticated Encryption)

`doca-aes-gcm` offloads AES-GCM (Galois/Counter Mode) encryption and decryption to the hardware, providing both confidentiality and authenticity (integrity) at line rate.

## 1. The AES-GCM Model: Authenticated Encryption
Unlike simple encryption, GCM produces both a **Ciphertext** and an **Authentication Tag**.

**Plaintext + Key + IV $\rightarrow$ [Hardware AES-GCM] $\rightarrow$ Ciphertext + Tag**

### 1.1. Key and IV Management
- **Key**: Must be provided in the correct length (128, 192, or 256 bits).
- **IV (Initialization Vector)**: Must be unique for every single encryption operation with the same key to prevent cryptographic failure.
- **AAD (Additional Authenticated Data)**: Optional data that is authenticated (included in the tag) but NOT encrypted.

### 1.2. Operation Modes
- **Encrypt**: Plaintext $\rightarrow$ Ciphertext + Tag.
- **Decrypt**: Ciphertext + Tag $\rightarrow$ Plaintext (fails if the tag does not match).

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Common Setup**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **Context Creation**: Create a `doca_aes_gcm` context.
3. **Capability Check**: Use `doca_aes_gcm_cap_*` to verify if the hardware supports the requested key length and block size.
4. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **Buffer Prep**: Register `doca_buf` for Plaintext, Ciphertext, Key, and Tag.
2. **Permissions**:
    - Source (Plaintext/Key): `DOCA_ACCESS_FLAG_LOCAL_READ_ONLY`.
    - Destination (Ciphertext/Tag): `DOCA_ACCESS_FLAG_LOCAL_READ_WRITE`.
3. **Task Submission**: Submit the encryption/decryption task to the `doca_pe`.
4. **Drain**: Call `doca_pe_progress()` until the completion event is received.
5. **Integrity Check (Decrypt only)**: Verify the return status of the task. If the tag check fails, the plaintext is corrupted or tampered with.

## 3. Safety & Error Taxonomy

### 3.1. Cryptographic Failures
- **Authentication Failure**: In decryption mode, if the tag is incorrect, the hardware returns an error. **Never use the output of a failed decryption.**
- **IV Reuse**: Using the same (Key, IV) pair for two different messages is a critical security vulnerability.

### 3.2. Common `DOCA_ERROR_*` Mapping
- **`DOCA_ERROR_INVALID_VALUE`**: Incorrect key length, unsupported IV size, or buffer alignment issues.
- **`DOCA_ERROR_NOT_PERMITTED`**: Memory permission mismatch on the `doca_buf`.

## 4. Path Selection: When NOT to use doca-aes-gcm
- **Non-GCM Modes**: For AES-CBC or AES-XTS $\rightarrow$ use a CPU library (OpenSSL).
- **Non-AES Algorithms**: For ChaCha20 or Poly1305 $\rightarrow$ use a CPU library.
- **Tiny Payloads**: For very small packets, the DMA setup overhead can exceed the benefit of hardware acceleration.

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md) for hardware-level crypto hangs.
