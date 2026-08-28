---
name: deepstream-sop
description: Deploy and operate the DeepStream SOP (Standard Operating Procedure) Inference Microservice for industrial assembly-line compliance.
owner: "windy@nvidia.com"
service: "deepstream-sop"
version: "1.0.0"
license: "CC-BY-4.0 AND Apache-2.0"
metadata:
  author: "Wind Yuan <windy@nvidia.com>"
  tags: ["deepstream", "sop", "vlm", "triton", "fastapi", "industrial-ai"]
---

# DeepStream SOP Inference Microservice

A GPU-accelerated FastAPI service designed to verify industrial assembly-line compliance. It uses a combination of event boundary detection (GEBD) and VLM classification to ensure operators perform steps in the correct sequence.

## 🎯 Purpose & Trigger
Use this skill when the user needs to build, deploy, or debug the SOP (Standard Operating Procedure) Microservice.

### Trigger Phrases:
- "Verify operator step sequence"
- "Detect missing or out-of-order SOP steps"
- "Run VLM-based SOP checking on industrial cameras"
- "Call /v1/chat/completions with a video file or RTSP stream"
- "Deploy DeepStream SOP microservice"

## 🏗️ Architecture Overview

The service is a hybrid pipeline combining DeepStream for efficient video processing and VLM for semantic reasoning.

| Component | Role | Technology |
| :--- | :--- | :--- |
| **Video Processor** | Event boundary detection (GEBD) | DeepStream + Triton CAPI |
| **SOP Engine** | Step sequence verification | FastAPI + VLM (Cosmos Reason 1/2) |
| **Ingestion** | RTSP, File, or Basler Cameras | GStreamer / Pylon |
| **Output** | Compliance alerts, event logs | Kafka (NvProto/JSON), SSE Streaming |

## 🚀 Operational Workflow

### 1. Deployment
Deploy using the provided `docker-compose.yml`. Ensure the environment has access to:
- **NVIDIA Container Toolkit**
- **Triton Inference Server** (for GEBD models)
- **VLM Backend** (via vLLM or similar)

### 2. Model Integration
The service uses specialized models for event detection:
- **GEBD Model**: Detects transition boundaries between different assembly steps.
- **VLM**: Classifies the action within the boundary to confirm the correct SOP step.

### 3. Inference & API Usage
The microservice exposes a FastAPI endpoint (`/v1/chat/completions`).
- **Input**: Video file, RTSP URL, or Basler camera ID.
- **Process**: Video $\rightarrow$ GEBD (Boundary Detection) $\rightarrow$ VLM (Step Verification) $\rightarrow$ Sequence Logic.
- **Output**: JSON response indicating compliance, missing steps, or order violations.

## ⚠️ Critical Technical Details

### 1. Basler/Pylon Integration
For industrial cameras, the service uses a Pylon wrapper.
- **Emulation**: Supports emulation mode for testing without physical hardware.
- **Config**: Ensure the Pylon driver is correctly mapped in the Docker container.

### 2. Latency Measurement
The service tracks **chunk-level latency**. Monitor the time taken from event boundary detection to VLM classification to ensure real-time feedback.

### 3. Output Formats
The service supports multiple output sinks:
- **SSE (Server-Sent Events)**: For real-time streaming of compliance status.
- **Kafka**: For integration with factory-level monitoring systems.

## 🛠️ Troubleshooting
- **No Detections**: Check if the GEBD model is correctly loaded in Triton.
- **Wrong Sequence**: Verify the SOP definition (the "Golden Sequence") matches the current assembly process.
- **Camera Timeout**: Check Basler Pylon connectivity and network stability.

## 🔀 Routing & Handoff
- **General DeepStream**: Handoff to `deepstream-generate-pipeline` for custom pipelines.
- **Performance**: Handoff to `deepstream-profile-pipeline` for latency tuning.
