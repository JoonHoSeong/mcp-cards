---
name: deepstream-generate-pipeline
description: Build DeepStream GStreamer pipelines interactively. Use when the user asks about pipelines for video/image inference, detection, tracking, or streaming.
owner: NVIDIA CORPORATION
service: deepstream
version: 1.0.0
reviewed: 2026-04-27
license: CC-BY-4.0 AND Apache-2.0
metadata:
  author: "NVIDIA CORPORATION"
  tags: ["deepstream", "gstreamer", "pipeline", "nvinfer", "nvstreammux", "nvtracker"]
---

# DeepStream Pipeline Builder

Generate ready-to-run `gst-launch-1.0` pipelines for NVIDIA DeepStream SDK by collecting pipeline requirements through an interactive questionnaire, then assembling the pipeline using a standalone BM25 retrieval backend with structural metadata boosting.

## 🎯 Purpose & Trigger

Use this skill to translate natural language requests into a technically valid DeepStream pipeline.

### Trigger Phrases:
- "Build a pipeline to infer on a video"
- "Detect and track objects in an RTSP stream"
- "Create a deepstream pipeline for 4 cameras"
- "Run inference on an image and save the result"
- "How do I build a pipeline with nvinfer and nvtracker?"

## 🛠️ Prerequisites

- **DeepStream SDK**: Installed at `/opt/nvidia/deepstream/deepstream/`
- **GStreamer**: `gst-launch-1.0` and `gst-inspect-1.0` on `PATH`
- **Platform**: x86 dGPU (T4, A100, L40, RTX) or aarch64 (Jetson Orin/Xavier/Nano)

## 🚀 The 7-Step Execution Workflow

### Step 1 — Collect Pipeline Requirements
You **MUST** read `references/requirement-extraction.md` first.
1. **Infer from query**: Identify which of the 7 parameters are already specified:
   - Input Source, Num Sources, Inference Mode, Tracker, Sink, Platform, Extras.
2. **Ask User**: Use `AskUserQuestion` for the unknowns. Do NOT silently default tracker/sink/platform.
3. **Confirm**: Quote the inferred parameters back to the user.

### Step 2 — Build the Natural Language Query
Construct a descriptive query string following this pattern:
`"Please provide a GStreamer pipeline that [operation] on [num_sources] [input_type] [input_detail] [tracker_detail] and [output_action] [platform_detail]"`

### Step 3 — Run the Pipeline Generator Script
Execute the backend script:
```bash
python3 <skill-path>/scripts/generate_pipeline.py \\
  --query "<constructed_query>" \\
  --source-type "<source_type>" \\
  --num-sources <N> \\
  --inference "<inference_mode>" \\
  --tracker "<tracker_type>" \\
  --sink "<sink_type>" \\
  --platform "<platform>" \\
  --extras "<extras>" \\
  --format compact
```
**Note**: Always use `--format compact` to get the top retrieved pipeline.

### Step 4 — Validate the Pipeline
Run the validation script to catch syntax and linking errors:
```bash
python3 <skill-path>/scripts/validate_pipeline.py "<assembled_pipeline>" --format summary
```
**Limit**: Max 2 retries for fixes. If it still fails, present as-is with a warning.

### Step 5 — Pre-flight Path Check
Run `ls` over the default paths (sample video, PGIE config, tracker lib) to verify they exist. If missing, prepend a `⚠` warning to the output.

### Step 6 — Present the Pipeline (STRICT FORMAT)
Your response **MUST** follow this exact structure:
1. **Status Badge**: `✓ Validated · 11 elements · 0 warnings · confidence: HIGH`
2. **Bash Block**: A **single line** `gst-launch-1.0 -e ...` command. No `\\`, no shell variables.
3. **Breakdown Table**: Grouped by stage (Source $\rightarrow$ Mux $\rightarrow$ Inference $\rightarrow$ Tracking $\rightarrow$ Composition $\rightarrow$ Render).
4. **Suggestions**: Bullet list (e.g., "Change number of streams", "Switch to file output").

### Step 7 — Offer Refinement
Ask if the user wants to adjust any parameters (e.g., "Change the number of streams?"). If so, return to Step 2.

## ⚙️ Pipeline Assembly Rules
If the script is unavailable or returns `confidence: low`, use the rules in `references/assembly-rules.md` to manually assemble the pipeline.

## ⚠️ Forbidden Anti-Patterns
- **NO** `cat > /tmp/pipeline.sh` or heredocs.
- **NO** shell variables like `${VAR}` in the output command.
- **NO** line continuations (`\\`) in the bash block.
- **NO** using the `Write` tool to create a script unless explicitly asked by the user.

## 🛠️ Troubleshooting & Testing
- **Element missing**: Run `gst-inspect-1.0 <element>` to verify installation.
- **Test Suite**: Run `python3 -m unittest discover -s <skill-path>/tests -v` to verify the retriever.
