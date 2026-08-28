---
name: vss-setup-behavior-analytics
description: Deploy the vss-behavior-analytics service standalone with custom entrypoints, configuration sources, and optional calibration. Supports runtime dynamic updates via Kafka.
license: Apache-2.0
metadata:
  author: "NVIDIA Video Search and Summarization team"
  version: "3.2.0"
  github-url: "https://github.com/NVIDIA-AI-Blueprints/video-search-and-summarization"
  tags: ["nvidia", "blueprint", "operational", "deployment", "behavior-analytics", "dynamic-config"]
---

# VSS Setup Behavior Analytics (Standalone)

This skill provides a comprehensive workflow for deploying the `vss-behavior-analytics` container as a standalone service. It focuses on the spatial-AI analytics pipeline, allowing operators to customize the entrypoint and configuration without deploying the entire warehouse blueprint stack.

## 🎯 Purpose & Use Case

The primary goal is to bring up the behavior analytics pipeline with specific operational parameters.

### Use this skill when:
- You need to "deploy behavior analytics" or "run behavior-analytics standalone".
- You want to change the **entrypoint** (e.g., `fusion_search`, `dev_example`, `analytics 3D`, `mv3dt`).
- You want to use a custom behavior-analytics configuration or calibration JSON.
- You need to perform **Dynamic Updates** (Configuration or Calibration) to a running container without restarting it.
- You want to point the analytics engine at a specific warehouse-3d/mv3dt config.

## 🛠️ Prerequisites

Ensure the following are ready before deployment. Gaps should be surfaced to the user immediately.

### 1. Environment & Tools
- **Repository**: Full checkout with `$VSS_APPS_DIR` pointing to `<repo>/deploy/docker/`. This is mandatory for volume binds in the compose files.
- **Docker Runtime**:
  - Docker Engine **28.3.3**
  - Docker Compose plugin **v2.39.1+**
- **NGC Credentials**: `$NGC_CLI_API_KEY` must be exported for pulling private images. Refer to [`references/ngc-api-key-registry-login.md`](references/ngc-api-key-registry-login.md).

### 2. Infrastructure (Kafka/Broker)
- **Broker-less Mode**: The container starts and runs normally without a broker.
- **Broker-enabled Mode**: A reachable Kafka, Redis Streams, or MQTT broker is **required** for Dynamic Configuration and Dynamic Calibration.
- **Behavior**: Without a broker, the Kafka client retries a bounded number of times; the app may exit and cycle (`Restarting (N)`) until a broker is reachable.

### 3. Assets
- **Custom Files**: If using custom config or calibration, ensure the JSON files are present on the host disk and paths are known.

## 🚀 Deployment Workflow

The core deployment logic is detailed in [`references/deploy-behavior-analytics-service.md`](references/deploy-behavior-analytics-service.md). Follow these steps in order:

### Step 1: Entrypoint Selection
Pick the appropriate entrypoint based on the required analytics capability:
- `analytics 2D` / `analytics 3D` / `mv3dt`
- `dev_example`
- `fusion_search`

### Step 2: Configuration Source
Decide where the configuration comes from:
- **Profile-shipped**: Use the defaults provided in the repository.
- **Custom**: Provide a path to a custom JSON configuration.

### Step 3: Calibration Setup (Optional)
Decide if a calibration file is needed:
- **Profile-shipped / Custom**: Provide the file path.
- **Dynamic**: Leave empty; the app will wait for a dynamic-calibration notification over the broker.

### Step 4: Deployment & Verification
Execute the `docker compose` command as specified in the reference guide. Verify the container status via `docker ps` and probe `/docs` or `/health` endpoints.

## ⚡ Technical Deep-Dive: Dynamic Updates (Runtime)

When a broker is reachable, you can modify the behavior of a running container without redeploying.

### 1. Dynamic Configuration
Allows per-key patches or full snapshot updates to the analytics engine.
- **Target**: Kafka topic `mdx-notification`, Key: `behavior-analytics-config`.
- **Headers**:
  - `event.type`: `upsert` (patch) | `upsert-all` (snapshot) | `request-config` | `ack`.
  - `reference-id`: `video-analytics-api-<uuid>`, `behavior-analytics-<uuid>`, or source literal (`kafka`/`redis`/`mqtt`).
- **Body**: `{"status": ..., "config": <patch>, "error": ...}`.
- **Process**: The listener validates the envelope $\rightarrow$ validates the payload $\rightarrow$ persists to disk $\rightarrow$ applies to all workers $\rightarrow$ sends ACK.
- **Reference**: See [`references/dynamic-config.md`](references/dynamic-config.md) for the full wire contract.

### 2. Dynamic Calibration
Updates sensor calibration, ROIs, tripwires, and homographies.
- **Target**: Kafka topic `mdx-notification`, Key: `calibration`.
- **Headers**:
  - `event.type`: `upsert-all` (snapshot) | `upsert` (per-sensor merge) | `delete` (per-sensor removal).
  - `timestamp`: ISO-8601 UTC.
- **Process**: Validated against the vendored AJV schema. If a violation occurs, a `calibration schema violation` warning is logged, and the update is dropped (the previous valid calibration remains).
- **Reference**: See [`references/dynamic-calibration.md`](references/dynamic-calibration.md).

## 🛠️ Troubleshooting

| Error | Likely Cause | Recommended Solution |
| :--- | :--- | :--- |
| REST call returns `Connection Refused` | Microservice not running | Probe `/docs` or `/health`; redeploy via `vss-deploy-profile` or this skill. |
| HTTP 401/403 from NGC pulls | Missing/expired `NGC_CLI_API_KEY` | `docker login nvcr.io` and re-export the key. |
| Container OOM / Model load fail | Insufficient GPU memory | Switch to a smaller variant or free GPUs via `docker compose down`. |
| Container stuck in `Restarting` | Broker unreachable | Verify Kafka/Redis/MQTT connectivity. The app will stabilize once the broker is reachable. |

## 🔀 Routing & Handoff

- **Full Stack Deployment**: If the user wants the entire UI/agent/perception stack, hand off to [`vss-deploy-profile`](../vss-deploy-profile/SKILL.md) with profile `warehouse` (or `alerts`).
- **Runtime Updates**: If the user wants to publish a config/calibration update, verify broker connectivity, then use the **Dynamic Updates** section above.
- **Behavior Tuning**: If the user wants to change incident types or ROI rules, point them to [`references/configuration.md`](references/configuration.md) before editing the JSON.
