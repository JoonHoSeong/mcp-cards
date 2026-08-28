---
name: vss-search-archive
description: Perform natural-language fusion search across archived video, and manage video/RTSP ingestion for search indexing. Uses Cosmos Embed1 embeddings and Elasticsearch.
license: Apache-2.0
metadata:
  author: "NVIDIA Video Search and Summarization team"
  version: "3.2.0"
  github-url: "https://github.com/NVIDIA-AI-Blueprints/video-search-and-summarization"
  tags: ["nvidia", "blueprint", "operational", "search", "archived-video", "fusion-search"]
---

# VSS Search Archive (Alpha)

This skill enables natural-language fusion search across archived video content using Cosmos Embed1 embeddings. It also manages the end-to-end ingestion pipeline for video files and RTSP streams to make them searchable.

> ⚠ **Alpha Feature**: This functionality is in alpha and is not recommended for production use.

## 🎯 Purpose & Use Case

The primary goal is to find specific events, objects, or actions across a large archive of video data using natural language queries.

### Use this skill when:
- You need to "Find all instances of [object/action]" (e.g., "Find all forklifts", "Show me people running").
- You want to search for specific events within a time window (e.g., "What happened between 8am and noon?").
- You need to "Ingest a video file for search" or "Add an RTSP stream for search indexing".
- You need to remove a specific video source and its associated embeddings from the search index.

## 🛠️ Prerequisites

### 1. Infrastructure & Deployment
- **Search Profile**: The VSS `search` profile must be running. 
  - **Verification**: Probe `http://${HOST_IP}:8000/docs` (Agent) and `http://${HOST_IP}:9200/` (Elasticsearch).
  - **Deployment**: If missing, deploy using `vss-deploy-profile -p search`.
- **Host IP**: A valid `$HOST_IP` for the VSS agent backend must be provided by the user. **Do not default to localhost.**

### 2. Tooling & Credentials
- **Tools**: `curl`, `jq`, and Docker must be available on the agent host.
- **Credentials**: `$NGC_CLI_API_KEY` and `$NVIDIA_API_KEY` must be exported for any required image pulls.

### 3. Source Management
- **VIOS Integration**: The `vss-manage-video-io-storage` skill is used to verify the existence of video sources before initiating search.

## 🚀 Ingestion Workflow (Required before Search)

A source must be ingested via the **Agent Backend** (not bare VIOS) to trigger the embedding pipeline (RTVI-CV $\rightarrow$ RTVI-Embed).

### 1. Video File Ingestion (Three-Step Flow)
1. **Get Upload URL**: `POST /api/v1/videos` with `filename`.
2. **Chunked Upload**: `POST` the file to the received URL using the `nvstreamer` protocol.
3. **Complete Ingestion**: `POST /api/v1/videos/${sensorId}/complete`. This triggers the embedding generation.
   - **Wait**: Search is only possible once the `/complete` response indicates `chunks_processed > 0`.

### 2. RTSP Stream Ingestion
`POST /api/v1/rtsp-streams/add` with `sensorUrl`, `name`, and credentials. 
- **Note**: Embedding generation runs in the background. Poll with a low-`top_k` query to verify readiness.

### 3. Source Deletion
`DELETE /api/v1/videos/<video_id>` or `DELETE /api/v1/rtsp-streams/delete/<name>`. This cleans up both VIOS storage and Elasticsearch embeddings.

## 🔍 Search Workflow

### Step 1: Input Resolution & Source Verification
1. **Verify `$HOST_IP`**: Ensure the endpoint is explicitly provided.
2. **Verify Source Existence**: 
   - Use `vss-manage-video-io-storage` to list sources.
   - **Match Found**: Proceed to Step 2.
   - **Match Not Found**: Stop. Ask the user if they want to ingest the missing source using the *Ingestion Workflow* above.

### Step 2: Execute Search
The default method is using the Agent REST API.

**Basic Query:**
```bash
curl -s -X POST http://${HOST_IP}:8000/generate \
  -H "Content-Type: application/json" \
  -d '{"input_message": "find all instances of forklifts"}' | jq .
```

**Advanced Control (Structured Request):**
Use the `messages` array to pass `search_source_type` (e.g., `"rtsp"`).

**Tuning Parameters (Plain-text steer in `input_message`):**
- `video sources`: Filter to specific cameras/sensors.
- `top k`: Max results (Default: 10).
- `minimum similarity`: Filter noise (e.g., 0.3).
- `critic usage`: VLM verification of results (Default: true).

### Step 3: Result Presentation & Verification
1. **Format**: Present as a professional `Video Search Results` inspection report.
2. **Verification**: 
   - Explain the results concisely.
   - **Critique**: If `critic results` are present, include "Criteria" and "Critic result" (confirmed/rejected/skipped) columns.
   - **Visual Audit**: If the user agrees, download the `screenshot_url` of top candidates to `/tmp` and verify visually.

## ⚡ Technical Deep-Dive: Fusion Search Logic

The VSS Agent employs three search behaviors based on the query decomposition:
1. **Attribute-only**: Only appearance attributes are found (e.g., "red jacket").
2. **Embed-only**: Only semantic embeddings are used (e.g., "forklifts").
3. **Fusion**: Combines both. Runs embed search first, then reranks results using attribute filtering.

## 🛠️ Troubleshooting

| Error | Likely Cause | Recommended Solution |
| :--- | :--- | :--- |
| REST call returns `Connection Refused` | Search profile not running | Probe `:8000/docs` and `:9200/`; redeploy via `vss-deploy-profile -p search`. |
| Zero matches for known object | Embedding not yet generated | Verify `/complete` response for files or wait for RTSP stream processing. |
| HTTP 401/403 on pulls | Missing `NGC_CLI_API_KEY` | `docker login nvcr.io` and re-export the key. |
| High noise / False positives | Similarity threshold too low | Increase `minimum similarity` (e.g., to 0.4 or 0.5) in the query. |

## 🔀 Routing & Handoff

- **Detailed Search Strategies**: Refer to [`references/discovery_modes.md`](references/discovery_modes.md).
- **Diagnostics**: Refer to [`references/troubleshooting.md`](references/troubleshooting.md).
- **Full Stack**: Handoff to `vss-deploy-profile` for `warehouse` or `alerts` profiles.
- **Cross-Reference**: Use `vss-query-analytics` to correlate search hits with incident data.
