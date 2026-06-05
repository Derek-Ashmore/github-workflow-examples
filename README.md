# github-workflow-examples

## Workflows Index

| Category | Workflow | Description |
|----------|----------|-------------|
| AI Automation | [RuFlo Hive Mind](.github/workflows/ruflo-hivemind.yml) | **Reusable** workflow: headless `hive-mind init` (hierarchical-mesh + Byzantine consensus), then `hive-mind spawn` with a queen-led strategic swarm on the caller repo, then commits and pushes changes (excluding root dot-directories). Requires `ANTHROPIC_API_KEY`. See [example caller](.github/workflows/ruflo-hivemind-example.yml). |
| | [RuFlo Hive Mind (example caller)](.github/workflows/ruflo-hivemind-example.yml) | `workflow_dispatch` sample that calls [RuFlo Hive Mind](.github/workflows/ruflo-hivemind.yml) in this repo; accepts an `instruction` input and forwards `ANTHROPIC_API_KEY`. Other repos can call the reusable workflow with `uses: <owner>/github-workflow-examples/.github/workflows/ruflo-hivemind.yml@<ref>`. |
| Containers | [Docker Pull Push Example](.github/workflows/dockerpush.yml) | Pulls a Docker image, tags it, and pushes it to GitHub Container Registry (GHCR) using authentication |
| Document Conversion | [Markdown to Word](.github/workflows/markdown-to-word.yml) | Converts a Markdown file (with optional Mermaid diagrams) to Microsoft Word (.docx) using Pandoc and pandoc-mermaid-filter; input filename is a workflow_dispatch parameter, files live in `docs/` |
| Release Management | [Semantic Release](.github/workflows/semantic-release.yml) | **Reusable** workflow: creates a semver GitHub Release from the latest release tag (default bump patch; callers may pass `minor` or `major`). First release is `v1.0.0` when none exists. Requires `contents: write`. See [example caller](.github/workflows/semantic-release-example.yml). |
| | [Semantic Release (example caller)](.github/workflows/semantic-release-example.yml) | `workflow_dispatch` sample that calls [Semantic Release](.github/workflows/semantic-release.yml) in this repo; accepts `bump` (patch/minor/major) and optional `release_notes`. Other repos can call the reusable workflow with `uses: <owner>/github-workflow-examples/.github/workflows/semantic-release.yml@<ref>`. |
| Secrets Management | [Secret Test Workflow](.github/workflows/secrettest.yaml) | Tests how GitHub secrets are handled in environment variables, including string interpolation |
| Workflow Control | [Skip Test Workflow](.github/workflows/skiptest.yaml) | Demonstrates how to conditionally skip jobs using workflow_dispatch inputs and handle skipped job dependencies |
