---
name: nemo-relay-instrument-context-isolation
description: Framework for managing independent NeMo Relay scope stacks and ancestry propagation across concurrent requests, async tasks, and worker pools.
license: Apache-2.0
metadata:
  kind: tool
  layer: phase-2
  domain: meta
---

# NeMo Relay Context Isolation Guide

This skill defines the standard for managing scope-stack ownership in concurrent or multi-tenant applications to prevent observability cross-contamination.

## 1. The Core Isolation Rule
**Each independent request, agent, or workflow must own its own scope stack.** 
Sharing a mutable stack across unrelated concurrent work leads to shared ancestry, scope-local middleware leaks, and corrupted telemetry.

## 2. The Scope Model
- **Root Scope**: Every stack begins with a root scope.
- **Hierarchy**: Pushed scopes form a parent-child tree. This hierarchy determines event parentage and the visibility of scope-local middleware.
- **Lifetime**: Scope-local registrations are tied to the owning scope's lifetime and are destroyed when the scope closes.
- **Marks**: Use `mark` events for checkpoints, retries, or interrupts that aren't full spans.

## 3. Language-Specific Isolation Patterns

| Language | Isolation Mechanism | Implementation Detail |
|---|---|---|
| **Python** | `contextvars` | Relies on task-local behavior via `get_scope_stack()`. |
| **Rust** | Explicit Ownership | Use `create_scope_stack()` and `set_thread_scope_stack(...)`. |
| **Go** | Goroutine Safety | Use `NewScopeStack()` and `ScopeStack.Run(...)`. |
| **Node.js** | Execution Path | Use `createScopeStack()` and `setThreadScopeStack(...)`. |

## 4. Common Failure Modes & Diagnostics
- **The Root UUID Collision**: Events from different requests appearing under one root UUID $\rightarrow$ *Cause: Shared scope stack.*
- **Middleware Leakage**: Scope-local middleware acting on the wrong request $\rightarrow$ *Cause: Missing isolation at worker/thread boundary.*
- **The "Empty Stack" Error**: Work running without an active scope $\rightarrow$ *Cause: Crossing async/worker boundaries without propagating the stack.*
- **Cross-Request Contamination**: Events from Request A appearing as children of Request B $\rightarrow$ *Cause: Incorrect ancestry propagation.*

## 5. Validation Checklist
- [ ] A fresh scope stack is created for every independent entry point (API request, Cron job, etc.).
- [ ] The stack is explicitly propagated when work leaves the current execution context (e.g., `asyncio.create_task` or Go routines).
- [ ] Scope-local middleware is used only for behavior that should not outlive the request.
- [ ] Worker pools are initialized with isolated stacks.
