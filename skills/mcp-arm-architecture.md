# Arm MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Arm MCP |
| **Category** | Architecture |
| **Official** | Official |
| **Source** | https://github.com/arm/arm-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @arm/mcp-server` |

## Description
Arm MCP server provides AI agents with access to Arm architecture documentation, optimization guides, intrinsics references, and migration tools. It enables developers to write optimized code for Arm-based platforms, understand performance characteristics, and migrate x86 codebases to Arm with guided assistance.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_docs` | Search Arm architecture and developer documentation |
| `get_optimization_guide` | Get performance optimization guidance for specific workloads |
| `analyze_code_for_arm` | Analyze code for Arm compatibility and optimization opportunities |
| `get_migration_guide` | Get x86-to-Arm migration guidance for specific patterns |
| `list_intrinsics` | List NEON/SVE intrinsics by category or operation |
| `get_performance_data` | Get performance characteristics for specific instructions |

## Configuration
```json
{
  "mcpServers": {
    "arm": {
      "command": "npx",
      "args": ["-y", "@arm/mcp-server"]
    }
  }
}
```

## Use Cases
1. Optimize AI inference code for Arm-based edge devices  2. Migrate CUDA kernels to work alongside Arm CPU code on Jetson  3. Select optimal NEON/SVE intrinsics for data preprocessing

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Jetson development | jetson-init-target | Optimize Arm CPU code on Jetson Orin/AGX platforms with NVIDIA GPU acceleration |
| Edge AI optimization | nim-agent-blueprint | Tune Arm CPU preprocessing alongside NIM GPU inference |
| DPU programming | doca-programming-guide | Optimize Arm code running on NVIDIA BlueField DPU Arm cores |

## Prerequisites
- None (public documentation, no authentication required)

## Security Notes
- Read-only access to public Arm documentation; no sensitive data exposure
- Code analysis is performed locally; source code is not transmitted externally
- Performance data is reference-only; always benchmark on target hardware

## References
- https://github.com/arm/arm-mcp
- https://developer.arm.com/documentation
- https://developer.arm.com/architectures/instruction-sets/intrinsics
