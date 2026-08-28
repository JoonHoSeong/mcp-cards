---
name: cuopt-multi-objective-exploration
description: Trace and interpret the Pareto frontier across competing objectives using repeated single-objective cuOpt solves (weighted-sum and ε-constraint).
license: Apache-2.0
metadata:
  author: "NVIDIA cuOpt Team"
  version: "26.08.00"
  github-url: "https://github.com/NVIDIA/cuopt"
  tags: ["nvidia", "cuopt", "multi-objective", "pareto", "epsilon-constraint", "tradeoff"]
---

# Multi-Objective Exploration

This skill provides a structured workflow for solving problems with multiple competing objectives (e.g., Cost vs. Service Level, Distance vs. Vehicle Count). Since cuOpt optimizes one objective per solve, this skill orchestrates a sequence of solves to trace the **Pareto Frontier**—the set of non-dominated solutions where one objective cannot be improved without degrading another.

## 🎯 Purpose & Use Case

Use this workflow when the user needs to balance two or more conflicting goals and does not have a predefined weighting for them.

### Trigger Phrases:
- "Balance cost and service level"
- "Find the tradeoff between distance and vehicle count"
- "Minimize cost as much as possible without hurting coverage"
- "I want a set of optimal options, not just one answer"

## 🛠️ Core Methodology: The Pareto Frontier

A solution **A dominates B** if A is at least as good on every objective and strictly better on at least one. The Pareto Frontier consists of all non-dominated solutions.

### The Fundamental Rule:
**Do not collapse a multi-objective problem into a single weighted number and report it as "the answer."** This makes a tradeoff decision *for* the user. Instead, trace the frontier and let the user choose the preferred point.

## 🚀 Execution Workflow

### Step 1: Define & Formulate Objectives
Before sweeping, each objective must be formulated correctly using `cuopt-numerical-optimization-formulation`. Ensure the sense (min/max) and scale are accurate.

### Step 2: Build a Payoff Table (Anchoring)
Solve each objective independently to find its absolute minimum/maximum.
- **Process**: For $k$ objectives, run $k$ separate solves.
- **Purpose**: 
  1. Sets the **sweep bounds** for the $\epsilon$-constraint method.
  2. Provides the **normalization scales** (essential for weighted-sum).

### Step 3: Choose a Scalarization Method

| Method | Mechanism | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **Weighted Sum** | $\text{min} \sum w_i f_i(x)$ | Trivial to implement; fast. | Only finds points on the **convex hull**; misses concave regions. |
| **$\epsilon$-Constraint** | $\text{min} f_1(x)$ s.t. $f_i(x) \le \epsilon_i$ | Recovers the **full frontier**, including concave regions. | Requires more solves (a grid over $\epsilon$ values). |

**Recommendation**: Use $\epsilon$-constraint when the problem is MILP or when completeness of the frontier is required.

### Step 4: Sweep, Collect, and Filter
1. **Sweep**: Iterate through a grid of weights or $\epsilon$ values.
2. **Warm-start**: Use the prior solution as a warm start for the next solve to reduce compute time.
3. **Filter**: Discard any dominated or duplicate points.
4. **Refine**: Use the **Dual Value** (for LP/QP) to identify the local slope. Where the slope jumps, refine the grid with more solves to capture the "knee" of the curve.

### Step 5: Gap Filling (for MILP/Non-convex)
If the swept frontier has large gaps (boxes larger than 3x the median), solve specific $\epsilon$-constraint subproblems targeted at the midpoints of those gaps.
- **Exact points**: Certified `Optimal` at the given gap.
- **Approximate points**: Time-limited incumbents (report the reported gap).

## ⚡ Technical Deep-Dive

### 1. The "Knee" Point
The knee is the region where the tradeoff is sharpest (a small gain in one objective requires a huge sacrifice in another). Highlight this point to the user as a balanced compromise, but do not auto-pick it.

### 2. Duals as Exchange Rates (LP/QP Only)
The dual value of a swept $\epsilon$-constraint represents the **local exchange rate** (slope).
- **Meaning**: "Increasing the budget by 1 unit improves the primary objective by $X$ units."
- **Diagnostic**: A zero dual indicates the bound is slack (the sweep has moved past the frontier).

### 3. Computational Budgeting
- **MILP Time-caps**: Always set a per-solve time limit on MILP sweeps to prevent the solver from over-spending on certifying optimality for a single point.
- **Resolution**: Start with a coarse grid $\rightarrow$ identify bends $\rightarrow$ refine locally.

## 🔀 Routing & Handoff

- **Formulation**: Handoff to `cuopt-numerical-optimization-formulation` for objective setup.
- **API Execution**: Use `cuopt-numerical-optimization-api` for the actual LP/MILP/QP solve calls.
- **Routing Problems**: The same frontier workflow applies to `cuopt-routing-api-python` (e.g., distance vs. vehicles).
