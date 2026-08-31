# Pandoc Universal Document Converter MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Pandoc Universal Document Converter MCP Server |
| **Category** | Media/Documents |
| **Official** | ⭐ Official (Pandoc MCP) |
| **Source** | https://github.com/pandoc/mcp-pandoc |
| **Transport** | stdio |
| **Install** | `npx -y @pandoc/mcp-server` |

## Tools & Capabilities
- `convert_document` — Convert between Markdown, PDF, DOCX, LaTeX, HTML, EPUB, and Org-mode
- `extract_citations` — Parse bibliography references (BibTeX, CSL JSON) in technical papers
- `render_pdf_with_typst` — Compile markdown to polished PDF using Typst/LaTeX engines
- `inspect_document_ast` — Output Pandoc JSON AST for structural document analysis

## Client Configuration
```json
{
  "mcpServers": {
    "pandoc": {
      "command": "npx",
      "args": [
        "-y",
        "@pandoc/mcp-server"
      ]
    }
  }
}
```

## Security & Best Practices
- Local conversion engine. Sandbox execution to prevent arbitrary shell command injection.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
