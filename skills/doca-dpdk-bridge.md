---
name: doca-dpdk-bridge
category: devops
description: Use when integrating DPDK-based data planes with DOCA libraries, enabling the use of DOCA accelerators (like DOCA Flow) within a DPDK application.
trigger: "User wants to bridge a DPDK port to a DOCA device, convert mbufs to DOCA buffers, or run a DPDK application that uses DOCA-side acceleration."
---

# DOCA DPDK Bridge (doca-dpdk-bridge) Skill Card

## Overview
The DOCA DPDK Bridge allows developers to leverage DOCA's hardware acceleration capabilities (such as DOCA Flow for steering and offloading) within an existing DPDK application. It provides a standardized mechanism to bind a DPDK-managed port to a `doca_dev`, allowing packets to flow between the DPDK data plane and DOCA's internal acceleration structures.

### Core Value Proposition
- **Acceleration Integration**: Brings DOCA's high-performance offloads into the DPDK ecosystem.
- **Seamless Buffer Conversion**: Provides utilities to convert between DPDK `rte_mbuf` and DOCA memory buffers.
- **Standardized Binding**: Establishes a formal link between a DPDK port ID and a DOCA device handle.
- **Hybrid Data Planes**: Enables "best of both worlds" architectures where DPDK handles general networking and DOCA handles specialized hardware acceleration.

## Implementation Path: The Bridge Lifecycle

### 1. Configuration & Setup (`## configure`)
1. **Five-Way Version Match**: Verify compatibility between (1) DOCA version, (2) BlueField firmware, (3) `doca-dpdk-bridge` library, (4) `libdpdk` version, and (5) `doca_dev` capabilities.
2. **DPDK Preconditions**: Ensure the target PCIe port is bound to a DPDK-compatible driver (e.g., `mlx5_core` with `vfio-pci`) using `dpdk-devbind.py`.
3. **Memory Setup**: Confirm that hugepages are correctly mounted and available for the DPDK EAL.
4. **DOCA Device Binding**: Identify the target `doca_dev` using `doca_caps --list-devs` that corresponds to the DPDK port.

### 2. Execution & Run (`## run`)
1. **DPDK Initialization**: Start the DPDK EAL and ensure the target port is operational (`rte_eth_dev_start`).
2. **DOCA Device Open**: Open the corresponding `doca_dev` and initialize the DOCA context.
3. **Bridge Registration**: Register the DPDK port with the bridge to establish the binding.
4. **Buffer Conversion**: Implement the mbuf $\leftrightarrow$ DOCA-buf conversion logic in the data path.
5. **PE Progress**: Call `doca_pe_progress()` frequently to drain completions from the DOCA side of the bridge.

### 3. Verification & Testing (`## test`)
**Follow the "Iterative Seam-Check" loop.**
1. **Binding Smoke**: Confirm the bridge resolves cleanly and no `DOCA_ERROR_*` is returned during registration.
2. **Traffic Flow Check**: Verify DPDK-side counters increment and the DOCA library receives corresponding packets.
3. **Conversion Sanity**: Ensure mbuf $\leftrightarrow$ DOCA-buf conversions succeed without `DOCA_ERROR_INVALID_VALUE`.
4. **Scale-Up Test**: Increase the packet rate and verify that the PE progress rate matches the submit rate to prevent drops.

## Critical Rules & Safety

### 1. The "DPDK-First" Lifecycle Rule
**The DPDK side must be fully operational before the bridge is registered.**
- **Rule**: Call `rte_eth_dev_start` before attempting to register the DPDK port with the bridge. Out-of-order calls return `DOCA_ERROR_BAD_STATE`.

### 2. The Matched-Pair Mandate
**The bridge is highly sensitive to the specific version pair of DOCA and DPDK.**
- **Rule**: Never assume the "latest" version of DPDK works. Strictly adhere to the matched-pair window declared in the bridge headers/documentation.

### 3. Steering-Bridge Distinction
**The bridge connects the data planes; it does NOT program the hardware steering.**
- **Rule**: If packets are not reaching the bridge despite a healthy binding, the issue is in the steering rules (route to `doca-flow`), not the bridge itself.

## Diagnostic Ladder (Overlay)

| Layer | Bridge Manifestation | Key Action |
| --- | --- | --- |
| **Layer 1 (Install)** | `pkg-config` fails to find `doca-dpdk-bridge`. | Check `/opt/mellanox/doca/infrastructure/lib/pkgconfig/` for the actual module name. |
| **Layer 2 (Version)** | Bridge loads, but operations return `NOT_SUPPORTED` or `BAD_STATE`. | Verify the FIVE-way match; specifically check `pkg-config --modversion libdpdk` vs the bridge window. |
| **Layer 3 (Runtime)** | `DOCA_ERROR_NOT_PERMITTED` during open/create. | Confirm `mlnx` group membership and DPDK hugepage/devbind preconditions. |
| **Layer 4 (Binding)** | `DOCA_ERROR_NOT_FOUND` during port registration. | Verify the DPDK port ID and ensure the port is bound under DPDK. |
| **Layer 5 (Program)** | `DOCA_ERROR_BAD_STATE` during registration. | Ensure the DPDK port was started (`rte_eth_dev_start`) before bridge registration. |
| **Layer 6 (Buffer)** | `DOCA_ERROR_INVALID_VALUE` during buffer conversion. | Verify mbuf is within a registered `doca_mmap` and the target buffer is sufficiently sized. |
| **Layer 7 (Driver)** | `DOCA_ERROR_DRIVER` or `dmesg` kernel errors. | Capture `dmesg` and `mlxconfig` output; route to `doca-setup`. |

## Command Appendix

### Bridge-Specific Invocations
Probe for structured helpers first. Fall back to manual commands if probes fail.

| Command | Purpose | Healthy Output |
| --- | --- | --- |
| `pkg-config --modversion doca-dpdk-bridge` | Bridge library version check. | Semver matching `doca_caps --version`. |
| `pkg-config --modversion libdpdk` | Installed DPDK version check. | Semver within the bridge's supported window. |
| `dpdk-devbind.py --status` | Port binding verification. | Target port is under a DPDK-compatible driver. |
| `cat /proc/meminfo \| grep Huge` | Hugepage availability check. | `HugePages_Total` and `HugePages_Free` are non-zero. |
| `doca_caps --list-devs` | DOCA device identification. | Device corresponds to the DPDK port. |
| `DOCA_LOG_LEVEL=trace ./<binary>` | Bridge lifecycle trace. | Trace-level lines for every bridge call and DOCA Core transition. |

## Deferred Topic Boundaries
- **DPDK Installation**: Installing DPDK, mounting hugepages, and binding ports is handled by upstream DPDK docs (route via `doca-public-knowledge-map`).
- **DOCA Installation**: Installing the DOCA SDK and libraries is handled by `doca-setup`.
- **Hardware Steering**: Programming the flow rules to send packets to the bridge is handled by `doca-flow`.
- **Native DOCA Networking**: For projects without DPDK lock-in, using `doca_eth` is recommended (route to `doca-eth`).
