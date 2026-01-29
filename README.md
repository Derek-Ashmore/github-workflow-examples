# github-workflow-examples

## Workflows Index

| Workflow | Description |
|----------|-------------|
| [Skip Test Workflow](.github/workflows/skiptest.yaml) | Demonstrates how to conditionally skip jobs using workflow_dispatch inputs and handle skipped job dependencies |
| [Secret Test Workflow](.github/workflows/secrettest.yaml) | Tests how GitHub secrets are handled in environment variables, including string interpolation |
| [Docker Pull Push Example](.github/workflows/dockerpush.yml) | Pulls a Docker image, tags it, and pushes it to GitHub Container Registry (GHCR) using authentication |
| [Markdown to Word](.github/workflows/markdown-to-word.yml) | Converts a Markdown file (with optional Mermaid diagrams) to Microsoft Word (.docx) using Pandoc and pandoc-mermaid-filter; input filename is a workflow_dispatch parameter, files live in `docs/` |