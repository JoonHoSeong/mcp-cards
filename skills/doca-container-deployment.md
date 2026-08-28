---
name: doca-container-deployment
description: Deploy DOCA service containers (Argus, DMS, Firefly, UROM) on BlueField using kubelet standalone mode and static-pod manifests.
license: Apache-2.0
metadata:
  kind: library
  author: "NVIDIA DOCA Team"
  version: "26.08.00"
  tags: ["nvidia", "doca", "bluefield", "dpu", "kubernetes", "kubelet", "container", "deployment"]
---

# DOCA Container Deployment Runtime

This skill manages the operational lifecycle of deploying DOCA service containers on a BlueField DPU. Unlike a full Kubernetes cluster, this environment uses **kubelet in standalone mode**, where the unit of deployment is a YAML pod-spec dropped into a specific static-pod manifests directory.

## 🎯 Purpose & Trigger

Use this skill when the user is deploying a packaged DOCA service container (e.g., Argus, DMS, Firefly, UROM) or diagnosing its lifecycle.

### Trigger Phrases:
- "How do I run my DOCA service on the BlueField?"
- "Where do I put the pod-spec YAML file?"
- "My pod is stuck in `ImagePullBackOff` or `CrashLoopBackOff`."
- "The container is `Running`, but the service isn't responding."
- "Can I run DMS and Firefly together on one BlueField?"

## 🚀 Deployment Architecture: Kubelet Standalone

The deployment follows a strict "Drop-and-Run" pattern:
1. **The Agent**: A `kubelet` standalone agent runs on the BlueField Arm.
2. **The Input**: The operator drops a YAML pod-spec into the **static-pod manifests directory**.
3. **The Action**: `kubelet` detects the file, schedules the pod locally, and pulls the image from NGC.
4. **The Result**: The DOCA service container starts and binds to the DPU hardware.

## 🛠️ Operational Workflow

### 1. The Deployment Loop
1. **Pre-check**: Ensure `doca-setup` has verified the environment (DOCA install, BFB version, permissions).
2. **Pod-Spec Drop**: Copy the service-specific YAML to the static-pod directory.
3. **Status Monitor**: Use `kubelet` status commands to track the pod's transition from `Pending` $\rightarrow$ `ContainerCreating` $\rightarrow$ `Running`.
4. **Log Verification**: Inspect the `ENTRYPOINT` logs for critical startup errors.
5. **Liveness Probe**: Execute a trivial probe (e.g., `curl` to a health port) to confirm the service is actually ready.

### 2. Smoke-before-Bulk Protocol (MANDATORY)
To prevent system instability, follow this sequence before putting the DPU under workload:
**Pod Status `Running`** $\rightarrow$ **Clean ENTRYPOINT Logs** $\rightarrow$ **Positive Liveness Probe**.
If any step fails, do NOT proceed to bulk traffic; fix the root cause first.

## 🔍 The 8-Layer Error Taxonomy (Debugging Matrix)

When a pod fails, diagnose using this layered approach from the "top" (syntax) to the "bottom" (hardware).

| Layer | Name | Symptom | Primary Check |
| :--- | :--- | :--- | :--- |
| 1 | **Pod-Spec Syntax** | Pod never appears in `kubelet` list | YAML linting / Path check |
| 2 | **Pod Scheduling** | Pod stuck in `Pending` | Resource quotas / Node constraints |
| 3 | **Image Pull** | `ImagePullBackOff` / `ErrImagePull` | NGC credentials / Network reachability |
| 4 | **Runtime** | `CrashLoopBackOff` / Immediate exit | `ENTRYPOINT` logs / Architecture mismatch |
| 5 | **Volume Mount** | Mount errors in logs | Host path existence / Permissions |
| 6 | **Network Policy** | `Running` but unreachable | Host firewall / Port mapping |
| 7 | **Version** | Version mismatch in logs | BFB version vs. Image version |
| 8 | **Host-level** | Kernel panic / Hardware hang | `dmesg` / BlueField OS logs |

## ⚠️ Safety Policy & Constraints

- **No Cluster-Ops**: Do NOT attempt to use `kubectl` against a cluster API server; this is standalone mode.
- **Restart Discipline**: If a pod is in a restart loop, clear the root cause (e.g., fix the YAML) before letting `kubelet` continue restarting.
- **Image Tags**: Use only verified image tags from the public DOCA Container Deployment Guide. Do NOT invent tags.

## 🔀 Routing & Handoff

- **Setup**: If DOCA is not installed or the environment is unhealthy, route to `doca-setup`.
- **Bare-Metal**: If the workload is a standalone binary rather than a container, route to `doca-bare-metal-deployment`.
- **Service Config**: For specific configuration schemas (e.g., how to configure Argus), route to the matching service skill: `doca-argus`, `doca-dms`, `doca-firefly`, or `doca-urom-svc`.
- **Knowledge Map**: For general documentation and public guides, route to `doca-public-knowledge-map`.
