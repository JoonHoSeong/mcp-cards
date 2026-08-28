---
name: nemoclaw-for-langchain-deep-agents-code
title: "NemoClaw for LangChain Deep Agents Code"
description: "Run open-source Deep Agents Code, tuned for Nemotron 3 Ultra, to plan, edit and test code with enterprise governance."
publisher: "nvidia"
type: blueprint
updated: "2026-07-08T14:59:55.674Z"
canonical: https://build.nvidia.com/nvidia/nemoclaw-for-langchain-deep-agents-code
source_url: https://build.nvidia.com/qc69jvmznzxy/nemoclaw-for-langchain-deep-agents-code.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# NemoClaw for LangChain Deep Agents Code

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `nemoclaw-for-langchain-deep-agents-code` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/nemoclaw-for-langchain-deep-agents-code](https://build.nvidia.com/nvidia/nemoclaw-for-langchain-deep-agents-code) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/nemoclaw-for-langchain-deep-agents-code.md](https://build.nvidia.com/qc69jvmznzxy/nemoclaw-for-langchain-deep-agents-code.md) |
| Source snapshot | [../raw/nemoclaw-for-langchain-deep-agents-code.md](../raw/nemoclaw-for-langchain-deep-agents-code.md) |
| Last updated by publisher | 2026-07-08T14:59:55.674Z |

**Summary:** Run open-source Deep Agents Code, tuned for Nemotron 3 Ultra, to plan, edit and test code with enterprise governance.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

## What is NemoClaw for LangChain Deep Agents Code?

LangChain Deep Agents Code (dcode) is an open source terminal coding agent built on the Deep Agents SDK. It is designed for complex software engineering tasks that require planning, file edits, command execution, test runs, and iterative review.

With NVIDIA NemoClaw, Deep Agents Code runs as a governed blueprint on NVIDIA Nemotron open models in an [NVIDIA OpenShell](https://build.nvidia.com/openshell) runtime environment. Teams can point it at codebases while keeping source code, credentials, execution, and network access under enterprise control.

![Details about NemoClaw with LangChain Deep Agents Code](https://assets.ngc.nvidia.com/products/api-catalog/nemoclaw/nemoclaw-for-langchain-deep-agents-code/diagram.png)

## Why use Deep Agents Code with NemoClaw?

### Built for long-running coding tasks

Deep Agents Code can break down complex engineering work, inspect a repository, make coordinated edits across files, run commands and tests, and return a diff for review. It is designed for multi-step engineering work like modernizing legacy codebases, upgrading dependencies and frameworks, repairing tests, and making coordinated code changes that need planning, execution, and review.

### Open model and open harness

The blueprint pairs NVIDIA Nemotron 3 Ultra with LangChain's open-source Deep Agents Code harness. Teams get a model-neutral coding agent architecture with the flexibility to adapt the harness, model, and runtime as their needs change.

### Governed by default

Deep Agents Code runs inside an OpenShell runtime environment with deny-by-default networking, approval controls, and an audit trail. Teams can allow the agent to work on sensitive codebases while keeping control over what resources it can access and execute.

### Persistent context, memory, and skills

Deep Agents manages long-running context through summarization, result offloading, persistent memory, skills, and subagents. Teams can encode repository conventions, migration playbooks, review standards, and domain-specific instructions so the agent improves on repeated work.
