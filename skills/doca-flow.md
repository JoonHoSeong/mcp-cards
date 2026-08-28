---
name: doca-flow
description: Program high-performance packet steering and match/action pipes on NVIDIA NICs/DPUs using the DOCA Flow library.
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
  Requires DOCA SDK installed. Authority: `pkg-config --modversion doca-flow`.
  Supported on BlueField DPU and ConnectX NICs.
---

# DOCA Flow (Packet Steering & Match-Action)

`doca-flow` allows developers to define complex packet-steering rules in hardware (NIC/DPU) using a **Match $\rightarrow$ Action $\rightarrow$ Forward** pipeline model.

## 1. The Flow Pipeline Model: Match-Action-Forward
DOCA Flow does not use a single flat rule list. It uses a **Pipe** structure.

**Packet $\rightarrow$ [Pipe Stage 1: Match $\rightarrow$ Action] $\rightarrow$ [Pipe Stage 2: Match $\rightarrow$ Action] $\rightarrow$ Forward Target**

### 1.1. Matcher Capabilities
Hardware-accelerated matching on:
- **L2/L3/L4 Headers**: MAC, VLAN, IP, TCP/UDP ports.
- **5-Tuple**: The canonical match for flow steering.
- **Custom Offsets**: Matching specific bytes in the payload (limited by hardware).

### 1.2. Action Capabilities
- **Forwarding**: Steer to a specific Queue, Representor, or Port.
- **Encap/Decap**: Add/Remove headers (e.g., VXLAN, Geneve).
- **Modification**: Rewrite header fields (e.g., NAT).
- **Counting**: Increment hardware counters per rule.

## 2. Operational Workflow

### 2.1. Configure (`configure`)
1. **Foundation**: Establish `doca_dev` $\rightarrow$ `doca_ctx` $\rightarrow$ `doca_pe` (via `doca-common`).
2. **Port Bring-up**: Initialize the Flow port. 
    - **CRITICAL**: Use `doca_flow_port_cfg_set_port_id()` and correctly set the device source (VNF mode vs. Host mode).
3. **Pipe Definition**: Define the pipeline stages (Matchers and Actions).
4. **Validation**: Validate the pipe spec *before* hardware commit to avoid `DOCA_ERROR_*`.
5. **Start**: `doca_ctx_start(ctx)`.

### 2.2. Execution (`run`)
1. **Entry Programming**: Create entries (rules) and associate them with a pipe.
2. **Commit**: Commit the entries to hardware.
3. **Traffic Test**: Send packets and verify the rule hit using hardware counters.
4. **Observation**: Read counters via `doca_flow_port_read_counters()`.

## 3. Safety & Error Taxonomy

### 3.1. Non-negotiable Constraints
- **Pipe Decomposition**: A single pipe stage can only express one logic step. For complex logic (e.g., "Tunnel select AND Egress port select"), decompose into **multiple pipes** (Classifier $\rightarrow$ Encap $\rightarrow$ Forward).
- **API Ground Truth**: Always verify `doca_flow_*` symbols against the installed header (`/opt/mellanox/doca/include/doca_flow.h`) as version differences are frequent.

### 3.2. Common `DOCA_ERROR_*` Mapping
- **`DOCA_ERROR_INVALID_VALUE`**: Pipe spec violation (e.g., unsupported match/action combination).
- **`DOCA_ERROR_NOT_PERMITTED`**: Permission issues on the representor or port.
- **`Failed to get hws cap`**: Typically a device-placement issue (wrong DPU Arm vs. Host x86 target).

## 4. Path Selection: When NOT to use doca-flow
- **Kernel-level Steering**: For `tc flower`, `iptables`, or `eBPF` $\rightarrow$ route to kernel-specific skills.
- **Raw DPDK**: For `rte_flow` without DOCA $\rightarrow$ route to DPDK skills.
- **Bulk Data Move**: For memory copies $\rightarrow$ use [`doca-dma`](doca-dma.md).

## 5. Routing & Dependencies
- **Foundation**: [`doca-common`](doca-common.md) (for `doca_buf` and `doca_pe`).
- **Build**: [`doca-programming-guide`](doca-programming-guide.md).
- **Debug**: [`doca-debug`](doca-debug.md) for hardware-level flow hangs.
