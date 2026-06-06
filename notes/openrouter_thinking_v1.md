# OpenRouter Enhancement — Thinking (v1)

This document records my reasoning while implementing the OpenRouter/DeepSeek
enablement for RuFlo. Open questions are tracked separately in
`notes/openrouter_questions_v1.md`.

## Goal

Provide a new **callable** workflow, patterned after `ruflo-hivemind.yml` and its
example caller, that lets the user execute RuFlo via Claude Code while routing all
LLM traffic to DeepSeek through OpenRouter's Anthropic-compatible gateway —
instead of using Anthropic's own models.

## Approach

I treated `ruflo-hivemind.yml` as the structural template and changed only what was
necessary to swap the model backend. Keeping the step sequence identical (validate →
checkout → setup Node → install Claude Code → headless wrapper → `ruflo init` →
`hive-mind init` → `hive-mind spawn` → commit & push) keeps the two workflows easy
to diff and maintain side by side.

The single conceptual change is the model backend. The user supplied the override
as a set of bash `export`s. In GitHub Actions the natural, least-surprising place
for process-wide environment is a **job-level `env:` block**, so every step
(including the `npx ruflo` invocations that launch Claude Code) inherits the
override automatically. This is cleaner than re-exporting inside each `run:` block
and avoids drift between steps.

## Mapping the bash override to `env:`

| Bash export | Workflow `env:` | Rationale |
|---|---|---|
| `ANTHROPIC_BASE_URL="https://openrouter.ai/api"` | literal | Used verbatim as provided. Flagged `/v1` as an open question. |
| `OPENROUTER_API_KEY=<key>` | `${{ secrets.OPENROUTER_API_KEY }}` | Requirement: key comes from a GitHub secret. |
| `ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"` | `${{ secrets.OPENROUTER_API_KEY }}` | Auth token must carry the OpenRouter key. |
| `ANTHROPIC_API_KEY=""` | `""` | Forces the client to honor the base URL rather than calling Anthropic directly. |
| `ANTHROPIC_MODEL` / `_OPUS_MODEL` / `_SONNET_MODEL` | `${{ inputs.model }}` | Primary tiers driven by the `model` input so it stays configurable. |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` / `CLAUDE_CODE_SUBAGENT_MODEL` | `deepseek/deepseek-flash` | Fast tiers hard-pinned, matching the override verbatim. |
| `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK=1` | `"1"` | Skip org check for OpenRouter fast-mode compatibility. |

### Why the primary/fast split

The override maps the heavyweight tiers (Opus/Sonnet) to one slug and the fast
tiers (Haiku/subagent) to another. I wired the primary tiers to the `model` input
(so future models slot in there) and left the fast tiers pinned to
`deepseek/deepseek-flash` exactly as instructed. Whether the fast tier should become
its own input later is captured as an open question rather than assumed.

## Requirement-by-requirement reasoning

- **API key from a GitHub secret** — declared `OPENROUTER_API_KEY` as a `required`
  secret on the callable workflow and mapped it into both the OpenRouter key and the
  Anthropic auth token. The validation step fails fast with an `::error::` annotation
  if it is missing or empty.
- **Model input, default DeepSeek, DeepSeek-only (for now)** — `model` defaults to
  `deepseek/deepseek-chat`. `workflow_call` inputs cannot be `choice`, so I enforced
  "only DeepSeek" on the callable side with a `case` allow-list that rejects any
  other value. The example caller expresses the same constraint UI-side as a `choice`
  with a single option. This makes "the only option" true today while leaving a
  one-line edit (extend the `case`) to add models later.
- **Output in the execution summary** — every RuFlo step appends its stdout/stderr to
  `$GITHUB_STEP_SUMMARY` using the same capture-with-mktemp pattern as
  `ruflo-hivemind.yml`. The swarm step additionally records the active base URL and
  primary model so a reader can confirm DeepSeek was used.
- **Instruction and RuFlo version inputs** — carried over unchanged: `instruction`
  required, `ruflo_version` optional defaulting to `3.5.80`.

## Design decisions and trade-offs

- **Job-level `env:` over per-step exports** — single source of truth, guaranteed to
  reach the Claude Code subprocess; the trade-off is that the override applies to all
  steps, which is desired here.
- **Fail-fast model validation** — surfaces an unsupported model immediately with a
  clear annotation instead of burning minutes and failing deep inside the swarm with
  an opaque provider error. The cost is one line to maintain per added model.
- **Mirroring the example-caller pair** — produced both the callable workflow and an
  example caller to match the existing `ruflo-hivemind` pair, so the OpenRouter
  variant is discoverable and runnable the same way. Whether to keep the example file
  is left as an open question.
- **Verbatim override values** — I deliberately did not "fix" `https://openrouter.ai/api`
  to `.../v1` or substitute different DeepSeek slugs, because the user provided exact
  values. Anything I was unsure about is documented as a question rather than silently
  changed.

## Risks / things to watch

These are the most likely failure points at first run, each tracked as an open
question in the companion file:

1. Base URL may need a `/v1` suffix for OpenRouter's Anthropic-compatible endpoint.
2. `deepseek/deepseek-flash` may not be a valid OpenRouter slug (could be a marketing
   name); the fast tier would then fail.
3. Secret naming (`OPENROUTER_API_KEY` vs. reusing `ANTHROPIC_API_KEY`) is a choice
   the user may want to revisit.
