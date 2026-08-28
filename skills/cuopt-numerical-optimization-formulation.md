---
name: cuopt-numerical-optimization-formulation
description: LP, MILP, QP — concepts, problem-text parsing, and formulation patterns (parameters, constraints, decisions, objective). Concepts only; no API.
license: Apache-2.0
metadata:
  author: "NVIDIA cuOpt Team"
  version: "26.08.00"
  tags: ["nvidia", "cuopt", "linear-programming", "milp", "qp", "formulation", "concepts"]
---

# Numerical Optimization Formulation

Concepts and workflow for going from a problem description to a clear mathematical formulation across LP, MILP, and QP. This skill focuses on **modeling**, not the API.

## 🎯 The Formulation Workflow

When the user describes a business problem in natural language, follow this 4-step sequence to build the model.

### Step 1: Identify Decisions (Variables)
Ask: *"What is the smallest unit of decision I need to make?"*
- **Binary**: $x \in \{0, 1\}$ (e.g., "Do I open this warehouse?")
- **Integer**: $x \in \{0, 1, 2, \dots\}$ (e.g., "How many trucks do I send?")
- **Continuous**: $x \in [0, \infty)$ (e.g., "How much fuel should I allocate?")

### Step 2: Define the Objective (The Goal)
What are we minimizing or maximizing?
- **Linear**: $Z = \sum c_i x_i$ (e.g., Total Cost = $\sum \text{cost\_per\_unit} \times \text{units}$)
- **Quadratic (QP)**: $Z = \frac{1}{2}x^T Q x + c^T x$ (e.g., Variance minimization in portfolio optimization).
- **Key Rule**: In cuOpt, QP only supports **minimization**. To maximize, minimize the negative.

### Step 3: Establish Constraints (The Rules)
What limits the decisions?
- **Hard Constraints**: Must be satisfied (e.g., "Cannot exceed total warehouse capacity").
- **Soft Constraints**: Preferred but can be violated at a penalty (e.g., "Try to keep drivers under 8 hours, but add a penalty cost for overtime").
- **Formulation Pattern**: All constraints must be linear (e.g., $\sum a_i x_i \le b$).

### Step 4: Map Parameters (The Data)
Identify the constant values provided by the user:
- **Costs/Coefficients**: $c_i, a_i$.
- **Bounds**: Upper and lower limits for each variable.
- **Right-Hand Side (RHS)**: The $b$ in $\sum a_i x_i \le b$.

## 🔍 Problem Type Identification Matrix

| Feature | LP | MILP | QP |
| :--- | :--- | :--- | :--- |
| **Objective** | Linear | Linear | Quadratic (Convex) |
| **Variables** | Continuous | Mixed (Int/Bin/Cont) | Continuous |
| **Constraints** | Linear | Linear | Linear |
| **Sense** | Min/Max | Min/Max | **Minimize Only** |
| **Complexity** | P (Fast) | NP-Hard (Slower) | P (Fast) |

## 🛠️ Common Formulation Patterns

### 1. The "Selection" (Binary)
- **Pattern**: $x_i \in \{0, 1\}$
- **Use**: Facility location, project selection, shift assignment.

### 2. The "Capacity" (Linear Inequality)
- **Pattern**: $\sum \text{usage}_i \times x_i \le \text{Capacity}$
- **Use**: Resource limits, budget caps.

### 3. The "Either-Or" (Big-M)
- **Pattern**: $Ax + My \le b$, where $y \in \{0, 1\}$ and $M$ is a very large constant.
- **Use**: Enabling/disabling a constraint based on a binary decision.

### 4. The "Balance" (Equality)
- **Pattern**: $\sum \text{Inflow} - \sum \text{Outflow} = 0$
- **Use**: Network flow, supply chain balance.

## 🔀 Routing & Handoff

- **API Execution**: Once formulated, hand off to `cuopt-numerical-optimization-api` for implementation.
- **Trade-off Analysis**: If the user has multiple objectives, hand off to `cuopt-multi-objective-exploration`.
- **Technical Implementation**: For C++/CUDA internals, refer to `cuopt-developer`.
