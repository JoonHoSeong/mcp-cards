---
name: dynamo-interconnect-check
description: Validate NIXL/UCX/NCCL interconnect readiness for disaggregated serving over RDMA/NVLink. Confirms KV transport health before trusting disagg deployment benchmarks.
version: 1.0.0
category: infra
---

# Dynamo Interconnect Check

The `dynamo-interconnect-check` skill provides a read-only validation suite to ensure that the high-performance transport layer (NIXL/UCX/NCCL) required for disaggregated serving is correctly configured and operational. It prevents the "silent fallback" scenario where a deployment passes endpoint smoke tests but suffers from severe performance degradation due to a broken RDMA/NVLink path.

## Invariants

- **Read-Only Authority**: This skill is strictly read-only. It never mutates the cluster, modifies recipes, or prints secrets.
- **Disagg-Specific Scope**: This skill is only relevant for **disaggregated** or **multi-node** deployments. Single-node aggregated deployments do not exercise the transport and should skip this check.
- **Substrate-First Validation**: Endpoint reachability $\neq$ Transport health. A successful API response does not prove that KV transfers are using the optimized RDMA path.
- **Non-Destructive Probing**: Missing tools (e.g., `ibstat`) are reported as `skipped`, not `fail`, to avoid false negatives in restricted container environments.

## Execution Pipeline

### 1. Transport Environment Audit
**Goal**: Verify that the recipe defines the necessary transport variables for disaggregated serving.

- **Action**: Run `python3 scripts/check_interconnect.py env <recipe_path>`.
- **Critical Variables**: Check for `UCX_TLS`, `UCX_NET_DEVICES`, and `NCCL_IB_HCA`.
- **Interpretation**: Missing variables here are warnings (they might be baked into the image). If missing, proceed to Node Capability checks to verify the actual runtime environment.

### 2. Node Capability Probe
**Goal**: Confirm the physical and driver-level readiness of the GPU node/pod.

- **Action**: Run `python3 scripts/check_interconnect.py node --namespace <ns> --pod <pod>`.
- **Verification Matrix**:
    - **InfiniBand**: Check for Active links and available devices.
    - **GPUDirect RDMA**: Verify `nvidia_peermem` module is loaded.
    - **GDRCopy**: Check for driver presence.
    - **NVLink**: Validate GPU topology for P2P capabilities.

### 3. NIXL Reachability Validation
**Goal**: Confirm that the NIXL transport is actually capable of performing transfers.

- **Action**: Run `python3 scripts/check_interconnect.py nixl --namespace <ns> --pod <pod>`.
- **Outcome**: This check identifies the presence of NIXL test tooling and provides the exact command required to run a pairwise prefill $\leftrightarrow$ decode transfer test.
- **Constraint**: A full transfer test requires two scheduled GPU pods on the fabric.

## Gotchas

- **Silent Fallback**: The most dangerous failure mode is when NIXL/UCX fails and the system falls back to a slow TCP/socket path. The system stays "up," but throughput collapses.
- **Image-Baked Vars**: If the `env` check fails but the `node` check passes, the transport variables were likely injected via the container image or a Kubernetes Operator rather than the recipe text.
- **`nvidia_peermem` Dependency**: Without the `nvidia_peermem` module, NIXL cannot use GPUDirect RDMA and will fall back to staged copies, significantly increasing latency.
- **Pod-Level Tooling**: `skipped` results for `ibstat` or `nvidia-smi` mean the worker image is too slim for probing. This is inconclusive; the agent should suggest using a debug pod or an image with the NIXL test harness.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Escalation Path |
| :--- | :--- | :--- | :--- |
| `env` check reports all critical vars missing | Vars are injected via Operator/Image | Run `node` check inside pod to see actual environment | `dynamo-recipe-runner` (Config check) |
| `node` check reports no Active IB link | Fabric down or HCA not provisioned | Check `kubectl describe node` for IB labels/GPU | Cluster Admin / Infrastructure Team |
| `nvidia_peermem` reported as missing | GPUDirect RDMA module not loaded | Check `lsmod | grep nvidia_peermem` | Cluster Admin (Load module) |
| `nixl` check finds no test tools | Worker image lacks NIXL harness | Verify image tag or deploy a debug pod | Image Build Pipeline |
| Agg works, but Disagg is slow/hangs | Transport fallback (RDMA $\rightarrow$ TCP) | Run `node` and `nixl` checks to find the break | `dynamo-troubleshoot` |
| Endpoint smoke passes, but benchmarks are low | Substrate mismatch or fabric congestion | Compare `node` capability results across all worker pods | `dynamo-interconnect-check` (Full audit) |

## Command Appendix

| Command | Context | Purpose | Healthy Output |
| :--- | :--- | :--- | :--- |
| `python3 scripts/check_interconnect.py env <path>` | Pre-deploy | Audit recipe for NIXL/UCX vars | `ok` for all disagg-critical variables |
| `python3 scripts/check_interconnect.py node ...` | Post-deploy | Probe IB/GPUDirect/NVLink | `ok` for Active links and `nvidia_peermem` |
| `python3 scripts/check_interconnect.py nixl ...` | Post-deploy | Validate NIXL test readiness | `ok` + provided transfer-test command |
