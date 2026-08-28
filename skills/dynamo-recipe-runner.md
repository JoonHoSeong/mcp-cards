---
name: dynamo-recipe-runner
description: Select, validate, patch, and deploy NVIDIA Dynamo Kubernetes recipes for model/backend/GPU bring-up. Manages the lifecycle from recipe selection to OpenAI-compatible smoke testing.
version: 1.0.0
category: infra
---

# Dynamo Recipe Runner

The `dynamo-recipe-runner` skill is the primary orchestration engine for deploying Dynamo workloads on Kubernetes. It transforms user intent (model, framework, GPU type) into a running endpoint by leveraging a structured library of pre-verified recipes.

## Invariants

- **Recipe-First Approach**: Never author new manifests from scratch. Always select the closest existing recipe from the `recipes/` tree and apply minimal patches.
- **Pre-Flight Validation**: No `kubectl apply` may be executed until `scripts/recipe_tool.py validate` returns a clean bill of health or the agent has explicitly patched the reported blockers.
- **Secret Isolation**: Hugging Face tokens and other credentials must NEVER be written to files or logs. They must be managed exclusively via Kubernetes Secrets (e.g., `hf-token-secret`).
- **Minimum-Diff Patching**: When modifying manifests, patch only the specific required values (e.g., `storageClassName`, image tags). Do not reformat or rewrite entire YAML files to avoid introducing regression.
- **Proof of Success**: A deployment is not "complete" until a successful `/v1/models` response and at least one chat completion are verified via port-forwarding.

## Execution Pipeline

### 1. Pre-Flight & Environment Audit
**Goal**: Ensure the operator machine and target cluster are ready.
- **Local Check**: Verify `git status` and `kubectl config current-context`.
- **Cluster Check**: Verify `StorageClass` existence, Node GPU availability, and the presence of the `hf-token-secret` in the target namespace.
- **Degraded Mode**: If `kubectl` is unreachable, the agent must transition to "Command Generation Mode"—producing the exact sequence of commands for the user to run—rather than simulating execution.

### 2. Recipe Selection & Validation
**Goal**: Find the optimal recipe and resolve configuration gaps.
- **Discovery**: Use `python3 scripts/recipe_tool.py list --query <model> --framework <fw> --mode <mode>` to find the best fit.
- **Deep Inspection**: Read the recipe's `README.md`, `deploy.yaml`, and `perf.yaml`.
- **Formal Validation**: Run `python3 scripts/recipe_tool.py validate <recipe_path>`.
- **Blocker Resolution**: Identify and patch missing `storageClassName`, stale image tags, or mismatched GPU counts before proceeding.

### 3. Deployment Sequence
**Goal**: Execute a staged rollout to ensure model availability.
1. **Model Cache**: Apply model-cache manifests $\rightarrow$ `kubectl wait` for `job/model-download` to reach `Complete` status.
2. **Workload**: Apply `deploy.yaml` to the target namespace.
3. **Readiness**: Monitor `dynamographdeployment` and pod status until all components are `Ready`.

### 4. Verification (Smoke Test)
**Goal**: Prove the endpoint is functional and OpenAI-compatible.
- **Access**: `kubectl port-forward svc/<deployment>-frontend 8000:8000`.
- **L1 Check**: `curl http://127.0.0.1:8000/v1/models` (Verify model registration).
- **L2 Check**: Perform one chat completion request.
- **Advanced**: If `dynamo-router-starter` is available, use its `check_router_health.py` for a comprehensive health audit.

## Gotchas

- **StorageClass Mismatch**: The most common `validate` failure. If the cluster lacks a default `StorageClass`, the `model-cache` PVC will remain `Pending` forever.
- **Stale Image Tags**: Recipes often use test images. Always verify the `image` field against the current production registry before deploying.
- **HF Secret Naming**: Recipes typically expect `hf-token-secret`. If the user's secret has a different name, the model download job will fail with 401/403 errors.
- **Frontend Port Confusion**: Ensure the port-forward maps the correct internal service port (usually 8000) to the local host.

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Escalation Path |
| :--- | :--- | :--- | :--- |
| `validate` reports missing StorageClass | No default `StorageClass` in K8s | `kubectl get storageclass` | Patch `storageClassName` in manifests |
| Model-cache job stuck in `Pending` | PVC unbound or HF secret missing | `kubectl describe pvc` $\rightarrow$ check events | Create/Rename `hf-token-secret` |
| Worker pods `ImagePullBackOff` | Stale image tag or missing pull secret | Check `kubectl describe pod` for registry errors | Patch image tag / Add pull secret |
| `/v1/models` returns 4xx/5xx | Frontend not ready or port-forward fail | Check pod logs for `frontend` $\rightarrow$ verify port | `dynamo-troubleshoot` |
| `kubectl` cluster unreachable | VPN down or Context mismatch | `kubectl config current-context` | Switch to "Command Generation Mode" |
| `deploy.yaml` applied but pods not starting | Resource quota exceeded (GPUs) | `kubectl describe pods` $\rightarrow$ check `Insufficient nvidia.com/gpu` | Adjust GPU requests in recipe |

## Command Appendix

| Command | Purpose | Healthy Output |
| :--- | :--- | :--- |
| `python3 scripts/recipe_tool.py list ...` | Enumerate recipes | A formatted table of matching recipes |
| `python3 scripts/recipe_tool.py validate <path>` | Pre-apply check | `OK` or a list of specific blockers |
| `kubectl wait --for=condition=Complete job/model-download` | Sync model fetch | Exit 0 (Job finished successfully) |
| `kubectl get dynamographdeployment` | Check custom resource status | `READY` state for the deployment |
| `curl http://127.0.0.1:8000/v1/models` | Endpoint smoke test | JSON list containing the deployed model |
