---
name: doca-devemu
category: devops
description: Use when implementing PCIe device emulation on a BlueField DPU to expose virtual devices (e.g., virtio-net, virtio-fs) to the host.
trigger: "User needs to emulate a PCIe device on a BlueField DPU or troubleshoot why an emulated device is not appearing on the host."
---

# DOCA Device Emulation (doca-devemu) Skill Card

## Overview
`doca-devemu` allows a BlueField DPU to act as a PCIe endpoint, emulating various device classes (such as VirtIO devices) to the host system. This enables the DPU to provide hardware-accelerated services to the host while appearing as a standard PCIe device.

### Core Value Proposition
- **Host Transparency**: The host sees a standard PCIe device, allowing it to use native kernel drivers.
- **DPU-Driven Logic**: The DPU controls the behavior of the emulated device, enabling flexible service implementation.
- **Hardware Acceleration**: Offloads complex device logic from the host to the DPU hardware.

## Implementation Path: The Emulation Workflow

### 1. Configuration & Preparation (`## configure`)
1. **Module Selection**: Identify the correct `doca-devemu` sub-library (e.g., for virtio-net or virtio-fs).
2. **Firmware Enablement**: Verify that the firmware-side emulation-type slot is enabled via `mlxconfig`. If not, enable it and reset the BlueField.
3. **Capability Audit**: Check `doca_caps` on the DPU to ensure the specific emulation class is supported.
4. **Host Driver Readiness**: Ensure the host kernel has the matching driver loaded for the emulated class.

### 2. Execution & Bring-up (`## run`)
1. **Context Initialization**: Create a sub-library context on the DPU.
2. **Device Activation**: Call `doca_ctx_start()` to expose the emulated device to the host.
3. **Host Enumeration**: Verify the device appears in the host's `lspci` output.
4. **Driver Binding**: Confirm the host kernel driver binds to the emulated device.
5. **DPU-Side Driving**: Use `doca_pe_progress()` on the DPU to handle doorbell registrations and device operations.

### 3. Verification & Smoke Testing (`## test`)
A successful emulation is verified by the **Host-DPU Handshake**:
1. **Enumeration**: `lspci` on host shows the correct device class.
2. **Bind**: `dmesg` on host shows the driver successfully bound to the device.
3. **Operation**: A basic I/O operation (e.g., sending a packet for virtio-net) triggers a corresponding reaction on the DPU.

## Layered Debugging (`## debug`)
When emulation fails, diagnose using the **Emulation-Specific Ladder**:
1. **Enumeration Failure**: If `lspci` is empty $\rightarrow$ Check firmware slots (`mlxconfig`) or DPU context start status.
2. **Binding Failure**: If `lspci` is OK but no driver binds $\rightarrow$ Check host kernel drivers or VirtIO feature negotiation in host `dmesg`.
3. **Reaction Failure**: If bound but no DPU reaction $\rightarrow$ Check `doca_pe_progress()` loop on the DPU.
4. **Privilege Failure**: If `DOCA_ERROR_NOT_PERMITTED` $\rightarrow$ Check DPU process privileges or firmware configuration.
5. **I/O Failure**: If `DOCA_ERROR_IO_FAILED` $\rightarrow$ Check host-side driver interactions in host `dmesg`.

## Critical Rules & Safety

### 1. The "Host-Driver-Attached" Rule
**Never destroy a DPU context while the host driver is still attached.**
- **Rule**: Ensure the host driver is unbound or the device is logically detached before tearing down the `doca-devemu` context to avoid `DOCA_ERROR_BAD_STATE` and host-side kernel noise.

### 2. The "Capability-First" Mandate
**Never assume a sub-library is supported without a probe.**
- **Rule**: Always run the sub-library-specific `doca_devemu_<sub>_cap_*` query against the active `doca_devinfo` before configuring payloads.

### 3. The "No-Invention" Rule
**Never manufacture shorthand names for modules or symbols.**
- **Rule**: Report literal module and symbol names found in the installed headers and `pkg-config` files.

## Command Appendix

### Emulation-Specific Invocations
| Purpose | Command (Example) | Healthy Indicator |
| --- | --- | --- |
| FW Slot Check | `mlxconfig -d <bdf> q` | Emulation class is enabled in current/next-boot config. |
| DPU Capability | `doca_caps --list-devs` | Device advertises support for the chosen emulation class. |
| Host Enum | `lspci` (Host side) | Device class appears in the PCIe tree. |
| Host Bind | `dmesg \| tail -n 80` (Host side) | Clean bind line for the matching kernel driver. |
| DPU Logs | `dmesg \| tail -n 40` (DPU side) | No repeated `mlx5` or device-emulation-driver errors. |
| DPU Trace | `DOCA_LOG_LEVEL=trace ./<binary>` | Trace-level lines on every lifecycle transition. |

## Deferred Topic Boundaries
- **Backend Logic**: Designing the actual storage/network logic behind the emulation is out of scope; use the public Device Emulation umbrella guide.
- **Host Driver Dev**: Writing host-side kernel drivers is outside the scope of DOCA; refer to Linux kernel documentation.
- **Env Setup**: For firmware resets or DOCA installation, route to `doca-setup`.
- **General Debugging**: For the cross-cutting debug ladder, route to `doca-debug`.
