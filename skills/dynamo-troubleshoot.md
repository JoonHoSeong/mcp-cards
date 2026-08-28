---
name: dynamo-troubleshoot
description: Diagnose failed or unhealthy Dynamo deployments. Classifies failures into problem buckets and provides a top-down diagnostic ladder to resolve K8s, Resource, and Runtime issues.
version: 1.0.0
category: infra
---

# Dynamo Troubleshoot

The `dynamo-troubleshoot` skill is the primary diagnostic engine for Dynamo deployments. Its core mission is to transform ambiguous "it's not working" reports into a precise **Problem Class**, a **Strongest Signal**, and a **Corrective Action**. It follows a strict "read-only first, fix one layer at a time" philosophy to avoid destabilizing the cluster during triage.

## Invariants

- **Read-Only Triage**: The skill is strictly read-only. It collects evidence, classifies the failure, and recommends patches, but it **never** mutates the cluster directly. All remediation commands are returned to the user for execution.
- **Secret Sanctity**: No Kubernetes secrets or Hugging Face tokens shall be collected, printed, or logged. Authentication failures are identified by their *symptoms* (e.g., 401/403 in logs) rather than by inspecting the tokens themselves.
- **Top-Down Layering**: Diagnosis must always proceed from the lowest (infrastructure) to the highest (application) layer. Fixing a pod crash before fixing a missing `StorageClass` is a failure of the diagnostic process.
- **Single-Point Fix**: Only one configuration change should be applied per iteration. After each fix, the agent must re-verify the current layer before moving deeper.

## Execution Pipeline

### 1. Evidence Collection (The Debug Bundle)
**Goal**: Capture a point-in-time snapshot of the deployment state without manual `kubectl` hunting.

- **Action**: Run `python3 scripts/collect_dynamo_debug_bundle.py --namespace <ns> [--deployment-name <name>]`.
- **Bundle Contents**: Pod statuses, Event logs, Job states, PVC bindings, and `DynamoGraphDeployment` custom resource status.
- **Scope Control**: For large namespaces, always use `--deployment-name` to prevent bundle bloat and noise.

### 2. Failure Classification
**Goal**: Map the strongest signal to a specific problem bucket.

- **Analysis**: Use the `failure-decision-tree.md` to classify the error into one of the following buckets:
    - **Platform/Cluster**: Node failures, network partitions, `kubectl` Forbidden.
    - **Namespace/Secret**: Missing `hf-token-secret`, incorrect namespace targets.
    - **Storage/Model**: Unbound PVCs, `model-download` job failures, disk pressure.
    - **Image/Runtime**: `ImagePullBackOff`, stale tags, architecture mismatch.
    - **Resources**: `Insufficient nvidia.com/gpu`, OOMKilled, CPU starvation.
    - **Orchestration**: `DynamoGraphDeployment` reconciliation loops, operator errors.
    - **Networking/API**: Frontend service errors, port-forward drops, 502/504 Gateway timeouts.
    - **Worker/Backend**: `CrashLoopBackOff`, CUDA errors, model load failures.

### 3. Top-Down Diagnostic Ladder
**Goal**: Systematically eliminate failure points from the bottom up.

1. **Layer 1 (Foundation)**: Check Namespace $\rightarrow$ StorageClass $\rightarrow$ Node GPU availability $\rightarrow$ HF Secret.
2. **Layer 2 (Provisioning)**: Check PVC state $\rightarrow$ `model-download` job completion.
3. **Layer 3 (Orchestration)**: Check `DynamoGraphDeployment` status $\rightarrow$ Operator events.
4. **Layer 4 (Runtime)**: Check Pod status $\rightarrow$ `describe pod` $\rightarrow$ Container logs.
5. **Layer 5 (Access)**: Check Frontend service $\rightarrow$ Port-forward connectivity.
6. **Layer 6 (Application)**: Verify `/v1/models` $\rightarrow$ Verify `/v1/chat/completions`.
7. **Layer 7 (Performance)**: Run benchmark jobs (ONLY after Layer 6 is green).

### 4. Remediation Proposal
**Goal**: Provide the smallest reversible fix.

- **Action**: Recommend a specific patch based on the classification.
    - *Example*: If Layer 2 is failing due to a missing StorageClass $\rightarrow$ Patch `storageClassName` in the model-cache manifest.
    - *Example*: If Layer 4 is failing due to `Insufficient gpu` $\rightarrow$ Reduce GPU requests in the recipe.
- **Verification**: After proposing a fix, define the exact success signal that proves the layer is now healthy.

## Gotchas

- **The "Forbidden" Trap**: If `kubectl` returns `Forbidden` on events/pods, the issue is a missing RBAC RoleBinding for the service account, not a failure of the Dynamo deployment itself.
- ** lazed Operator**: If the `DynamoGraphDeployment` status is empty or stagnant, the `dynamo-platform` operator may be crashed or not watching the namespace.
- **The `Pending` Loop**: A `model-download` job stuck in `Pending` is almost always a PVC binding issue or a missing `hf-token-secret`. Do not restart the job without fixing the PVC/Secret first.
- **Symptom Overlap**: A `CrashLoopBackOff` in the worker can be caused by either an image mismatch (Layer 4) or a missing model file on the PVC (Layer 2). Always check the PVC status before analyzing the pod logs.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Next Step / Fix |
| :--- | :--- | :--- | :--- |
| `kubectl` returns Forbidden | Missing RBAC permissions | Check `kubectl auth can-i get pods` | Request read-only RoleBinding |
| `DynamoGraphDeployment` status empty | Operator not running | `kubectl get pods -n dynamo-system` | Restart `dynamo-platform` operator |
| `model-download` job `Pending` | PVC unbound or Secret missing | `kubectl describe pvc` $\rightarrow$ check events | Fix `storageClassName` or create `hf-token-secret` |
| Worker pods `CrashLoopBackOff` | GPU unavailable or Image mismatch | `kubectl describe pod` $\rightarrow$ check events | Check `nvidia.com/gpu` allocatable on nodes |
| `/v1/models` returns empty | Worker-to-Router registration fail | Check worker logs for etcd/NATS connection | `dynamo-troubleshoot` (Worker layer) |
| `/v1/chat/completions` times out | Worker backend hang or Fabric break | Check `dynamo-interconnect-check` | `dynamo-interconnect-check` |

## Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `python3 scripts/collect_dynamo_debug_bundle.py --namespace <ns>` | Full Triage Bundle | A directory containing all logs/events/status |
| `kubectl describe pod <pod> -n <ns>` | Runtime Error Check | `Running` state; no `BackOff` or `OOMKilled` events |
| `kubectl get pvc -n <ns>` | Storage Verification | `BOUND` state for all model-cache PVCs |
| `kubectl get events -n <ns} --sort-by='.lastTimestamp'` | Event Timeline | No `FailedScheduling` or `FailedMount` errors |
