# github-workflow-examples

## Workflows Index

| Workflow | Description |
|----------|-------------|
| [Skip Test Workflow](.github/workflows/skiptest.yaml) | Demonstrates how to conditionally skip jobs using workflow_dispatch inputs and handle skipped job dependencies |
| [Secret Test Workflow](.github/workflows/secrettest.yaml) | Tests how GitHub secrets are handled in environment variables, including string interpolation |
| [Docker Pull Push Example](.github/workflows/dockerpush.yml) | Pulls a Docker image, tags it, and pushes it to GitHub Container Registry (GHCR) using authentication |
| [Markdown to Word](.github/workflows/markdown-to-word.yml) | Converts a Markdown file (with optional Mermaid diagrams) to Microsoft Word (.docx) using Pandoc and pandoc-mermaid-filter; input filename is a workflow_dispatch parameter, files live in `docs/` |
| [RuFlo Hive Mind](.github/workflows/ruflo-hivemind.yml) | **Reusable** workflow: runs `npx ruflo hive-mind spawn` with a queen-led strategic swarm on the caller repo, then commits and pushes changes (excluding root dot-directories). Requires `ANTHROPIC_API_KEY`. See [example caller](.github/workflows/ruflo-hivemind-example.yml). |