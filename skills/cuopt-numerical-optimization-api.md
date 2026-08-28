---
name: cuopt-numerical-optimization-api
description: LP, MILP, and QP (beta) solver interfaces for cuOpt — supporting Python, C, and CLI.
license: Apache-2.0
metadata:
  author: "NVIDIA cuOpt Team"
  version: "26.08.00"
  github-url: "https://github.com/NVIDIA/cuopt"
  tags: ["nvidia", "cuopt", "linear-programming", "milp", "qp", "python", "c-api", "cli"]
---

# cuOpt Numerical Optimization API

This skill provides the technical interfaces to model and solve Linear Programming (LP), Mixed-Integer Linear Programming (MILP), and Quadratic Programming (QP) problems using NVIDIA cuOpt's GPU-accelerated solver.

## 🛠️ Interface Selection

Select the appropriate reference based on the user's environment:

| Interface | When to use | Reference |
| :--- | :--- | :--- |
| **Python** | User is writing Python code | [`references/python_api.md`](references/python_api.md) |
| **C / C++** | User is embedding in a C/C++ application | [`references/c_api.md`](references/c_api.md) |
| **CLI** | User is solving from MPS files via terminal | [`references/cli_api.md`](references/cli_api.md) |

**Pro Tip**: If the user is already using a modeling language like **AMPL, GAMS, Pyomo, or CVXPY**, cuOpt can act as the solver backend. Prefer this over porting the model to the cuOpt API.

## 🔍 Problem Type Classification

Correct classification is critical for performance and feasibility.

| If Objective is... | And Variables are... | Use | Logic |
| :--- | :--- | :--- | :--- |
| Linear | All Continuous | **LP** | Fastest, strongest optimality guarantees. |
| Linear | Mixed (Int/Binary/Cont) | **MILP** | For counts, yes/no decisions, or discrete assignments. |
| Quadratic | Continuous | **QP** | For variance, squared error, or portfolio optimization. |

### Variable Type Cheat Sheet:
- **"How many things?"** (Workers, Trucks, Machines) $\rightarrow$ **INTEGER**.
- **"Yes/No?"** (Open facility, Assign shift) $\rightarrow$ **BINARY** (Integer, lb=0, ub=1).
- **"How much?"** (Dollars, Hours, Tonnes, Rates) $\rightarrow$ **CONTINUOUS**.

## ⚡ QP Specific Rules (Beta)

1. **Minimize Only**: cuOpt QP only supports minimization. To maximize $f(x)$, minimize $-f(x)$ and negate the resulting objective value.
2. **Continuous Only**: Integer QP is not supported.
3. **Positive Semi-Definite (PSD)**: The matrix $Q$ must be PSD for a convex, well-posed problem.

## 📊 Dual Values & Sensitivity

Duals and reduced costs provide insights into the "value" of constraints.

- **Available for**: **LP and QP only**.
- **Not Available for**: **MILP** (integer optima are not continuous) and **Quadratic Constraints**.
- **PDLP Warmstart**: Only supported for **LP**. MILP does not accept PDLP warmstarts.

## 🛠️ Common Issues & Troubleshooting

| Issue | Likely Cause | Recommended Fix |
| :--- | :--- | :--- |
| **Infeasible** | Contradictory constraints (e.g. $x > 10$ and $x < 5$) | Review constraints; check for typo in signs or bounds. |
| **Unbounded** | Objective can go to $\infty$ without hitting a constraint | Check if a variable is missing a bound (e.g. missing upper bound on a max problem). |
| **Slow convergence** | Poorly scaled data (e.g. coefficients vary by $10^6$) | Scale constraints/objective to be closer to $[0.1, 10]$ range. |
| **Numerical instability** | Very large coefficients or near-singular matrices | Check for redundant constraints; use double-precision if supported. |

## 🔀 Routing & Handoff

- **Formulation**: Handoff to `cuopt-numerical-optimization-formulation` to define the mathematical model.
- **Exploration**: Handoff to `cuopt-multi-objective-exploration` for Pareto frontiers.
- **Developer**: Handoff to `cuopt-developer` for solver internals and build/test.
