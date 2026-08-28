---
name: dynamo-router-starter
description: Start, patch, and validate Dynamo router modes (round-robin, KV-aware, etc.) for optimized request distribution. Ensures endpoint health via OpenAI-compatible smoke tests.
version: 1.0.0
category: infra
---

# Dynamo Router Starter

The `dynamo-router-starter` skill manages the configuration and validation of the Dynamo frontend's routing logic. It allows the agent to quickly transition between different routing strategies to optimize for latency, throughput, or KV cache reuse across a cluster of workers.

## Invariants

- **Worker-First Dependency**: The router cannot route to nothing. A minimum of one worker must be registered and healthy (verified via `/v1/models`) before any routing mode change is considered successful.
- **Smoke $\neq$ Benchmark**: The `check_router_health.py` script provides a functional smoke test (Liveness/Readiness). It is NOT a performance benchmark. Any claims regarding throughput or latency improvements across modes must be backed by `dynamo-benchmark`.
- **KV-Event Authority**: KV-aware routing depends on workers publishing cache events. If workers are not configured to publish events, the router MUST be set to **Approximate KV Mode** (`DYN_ROUTER_USE_KV_EVENTS=false`) to prevent request hangs.
- **Mode-Isolation for Comparison**: When comparing routing modes (e.g., Round-Robin vs. KV), all other variables—model, worker count, prompt set, concurrency, and sampling—must remain identical.

## Execution Pipeline

### 1. Baseline Establishment
**Goal**: Get a functional endpoint running with the simplest routing logic.

- **Local Setup**: Run `python3 -m dynamo.frontend --router-mode round-robin --http-port 8000`.
- **Kubernetes Setup**: Identify the frontend service in the `deploy.yaml` of the active recipe. If not deployed, route to `dynamo-recipe-runner`.
- **Verification**: Confirm at least one model is discoverable via `/v1/models`.

### 2. Routing Mode Optimization
**Goal**: Enable advanced routing based on the workload (e.g., KV-reuse for chat).

- **Mode Selection**:
    - `round-robin`: Default, equal distribution.
    - `kv`: Optimized for KV cache reuse (requires worker events).
    - `least-loaded`: Dynamic load balancing.
    - `device-aware-weighted`: Hardware-specific optimization.
- **Application**:
    - **Local**: Update `--router-mode` flag.
    - **Kubernetes**: Patch the frontend service environment variable `DYN_ROUTER_MODE`.
- **Fallback Logic**: If KV routing is enabled but workers don't publish events, patch `DYN_ROUTER_USE_KV_EVENTS` to `"false"` to enable approximate mode.

### 3. Endpoint Smoke Validation
**Goal**: Prove the routing configuration is functional and OpenAI-compatible.

- **Action**: Run `python3 scripts/check_router_health.py --base-url http://127.0.0.1:8000`.
- **Success Criteria**:
    1. `/v1/models` returns a non-empty list.
    2. At least one `/v1/chat/completions` request completes successfully.
- **Failure Handling**: If the endpoint is unhealthy or workers are missing, immediately escalate to `dynamo-troubleshoot`.

## Gotchas

- **KV-Mode Hangs**: The most common failure in `kv` mode occurs when the router waits for cache events that never arrive from the workers. The solution is either to fix worker config or switch to approximate mode.
- **Port-Forward Drops**: In Kubernetes, `kubectl port-forward` can silently drop. If a `Connection Refused` error occurs, re-verify the service name and restart the forward before diagnosing the router.
- **Empty Model List**: If `/v1/models` is empty, the router is healthy but the worker-to-router registration (via etcd/NATS) has failed.
- **False Throughput Claims**: A single successful chat request in KV mode does not prove a throughput increase. This is a "smoke success," not a "performance win."

## Diagnostic Ladder

| Symptom | Likely Cause | Verification Action | Escalation Path |
| :--- | :--- | :--- | :--- |
| `/v1/models` returns empty list | Worker registration failure | Check worker pods $\rightarrow$ verify etcd/NATS connectivity | `dynamo-troubleshoot` |
| Smoke chat request times out | Router healthy, workers failing | Inspect worker logs for request processing errors | `dynamo-troubleshoot` |
| KV mode hangs / high latency | Missing worker KV events | Check worker config $\rightarrow$ set `DYN_ROUTER_USE_KV_EVENTS=false` | Worker Config / Infra Team |
| Connection refused (K8s) | Port-forward dropped | Re-run `kubectl port-forward` $\rightarrow$ check svc name | `dynamo-recipe-runner` (Svc check) |
| `/v1/models` works, but chat fails | Router-to-Worker transport break | Check `dynamo-interconnect-check` for fabric health | `dynamo-interconnect-check` |

## Command Appendix

| Command | Context | Purpose | Healthy Output |
| :--- | :--- | :--- | :--- |
| `python3 -m dynamo.frontend --router-mode <mode> ...` | Local | Start frontend with specific routing | Process starts; port 8000 opens |
| `kubectl patch ... DYN_ROUTER_MODE=<mode>` | Kubernetes | Update routing strategy in-cluster | Patch applied successfully |
| `python3 scripts/check_router_health.py --base-url <url>` | Validation | Smoke-test endpoint and model access | `OK`: Models found $\rightarrow$ Chat success |
