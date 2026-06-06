# Documentation Improvement Suggestion

## Finding

The `merge-to-main.yml` workflow exists in `.github/workflows/` but is **not listed**
in the README's Workflows Index table.

### What `merge-to-main.yml` does

It runs on every push to `main` and invokes `semantic-release.yml` with a `patch`
bump — effectively auto-releasing on every merge to main.

### Current README index (12 workflows, 1 missing)

| Category | Workflow | In README? |
|---|---|---|
| AI Automation | `ruflo-hivemind.yml` | Yes |
| AI Automation | `ruflo-hivemind-example.yml` | Yes |
| AI Automation | `ruflo-openrouter.yml` | Yes |
| AI Automation | `ruflo-openrouter-example.yml` | Yes |
| Containers | `dockerpush.yml` | Yes |
| Document Conversion | `markdown-to-word.yml` | Yes |
| Release Management | `semantic-release.yml` | Yes |
| Release Management | `semantic-release-example.yml` | Yes |
| Secrets Management | `secrettest.yaml` | Yes |
| Workflow Control | `skiptest.yaml` | Yes |
| **Workflow Control** | **`merge-to-main.yml`** | **No** ← missing |

## Suggested fix

Add a row to the README's Workflows Index table under a new or existing category.
Recommended placement: under "Release Management" (since it triggers semantic-release)
or under a new "Workflow Control" row next to `skiptest.yaml`:

```markdown
| Workflow Control | [Merge to Main](.github/workflows/merge-to-main.yml) | Runs on push to `main`; triggers `semantic-release` with a patch bump for automatic releases on merge |
```

## Rationale

- Every other `.github/workflows/*.yml` file is documented in the README index.
- The merge-to-main workflow is a key piece of the release pipeline (it automates
  semantic-release on every main push), so discoverability matters.
- The README explicitly aims to be a complete index ("Workflows Index"), so the
  omission is a defect in that goal.