# OpenRouter Enhancement — Notes, Thinking & Questions (v1)

This document records my thinking while implementing the OpenRouter/DeepSeek
enablement for RuFlo, plus open questions for you.

## What I built

1. `.github/workflows/ruflo-openrouter.yml` — a new **callable** (`workflow_call`)
   workflow patterned after `ruflo-hivemind.yml`. It is identical in structure
   (validate → checkout → setup Node → install Claude Code → headless wrapper →
   `ruflo init` → `hive-mind init` → `hive-mind spawn` → commit & push) but adds
   the OpenRouter/DeepSeek environment override at the job level so Claude Code
   routes all model calls to DeepSeek behind the scenes.

2. `.github/workflows/ruflo-openrouter-example.yml` — an example caller patterned
   after `ruflo-hivemind-example.yml`, wired to `workflow_dispatch`.

## How the requirements were met

- **OpenRouter API key from a GitHub secret** → the callable workflow declares a
  required secret `OPENROUTER_API_KEY` and maps it into both `OPENROUTER_API_KEY`
  and `ANTHROPIC_AUTH_TOKEN`.
- **Model input, defaulting to DeepSeek and DeepSeek-only** → `model` input
  defaults to `deepseek/deepseek-chat`. The callable workflow's validation step
  rejects any other value (fail-fast) so it is effectively the only option today,
  while leaving room to add more later. The example caller exposes it as a
  `choice` with a single option (`workflow_call` inputs cannot be `choice`, so the
  callable side enforces the constraint via validation instead).
- **Output noted in the execution summary** → every RuFlo step writes stdout/stderr
  to `$GITHUB_STEP_SUMMARY`, exactly like `ruflo-hivemind.yml`. The swarm step also
  notes which model/base URL was used.
- **Instruction and RuFlo version inputs** → carried over unchanged from
  `ruflo-hivemind.yml` (`instruction` required, `ruflo_version` default `3.5.80`).

## The DeepSeek override (translated to GitHub Actions `env:`)

The provided bash `export`s were translated to job-level `env:` entries:

| Bash export | Workflow env |
|---|---|
| `ANTHROPIC_BASE_URL="https://openrouter.ai/api"` | same |
| `OPENROUTER_API_KEY=<key>` | `${{ secrets.OPENROUTER_API_KEY }}` |
| `ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"` | `${{ secrets.OPENROUTER_API_KEY }}` |
| `ANTHROPIC_API_KEY=""` | `""` |
| `ANTHROPIC_MODEL`, `..._OPUS_MODEL`, `..._SONNET_MODEL` | `${{ inputs.model }}` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL` | `deepseek/deepseek-flash` |
| `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK=1` | `"1"` |

I mapped the **primary** tiers (Opus/Sonnet) to the `model` input and left the
**fast** tiers (Haiku/subagent) hard-pinned to `deepseek/deepseek-flash`, matching
the override instructions verbatim.

## Open questions for you

1. **Base URL path.** The instructions specify `https://openrouter.ai/api`.
   OpenRouter's documented Anthropic-compatible endpoint is sometimes written as
   `https://openrouter.ai/api/v1`. I used exactly what you provided. Should the
   base URL include `/v1`? (If the swarm step fails auth/404s, this is the first
   thing to check.)

2. **Secret name.** I named the secret `OPENROUTER_API_KEY`. Is that the name you
   want callers to use, or should it stay `ANTHROPIC_API_KEY` for drop-in
   compatibility with the existing example?

3. **Fast/Haiku model.** I hard-pinned `deepseek/deepseek-flash` for the Haiku and
   subagent tiers, per your instructions. Do you want this exposed as its own input
   (e.g. `fast_model`) when you add more models later, or keep it coupled to the
   primary `model` choice?

4. **`deepseek/deepseek-flash` availability.** Please confirm `deepseek/deepseek-flash`
   is a valid OpenRouter model slug. I could not verify it from here; if it is not
   listed, the fast tier calls may fail. (The override instructions reference
   "DeepSeek Flash" and "DeepSeek Pro (Chat/Reasoning v4)" — these may be marketing
   names rather than exact OpenRouter slugs.)

5. **Model validation strictness.** I made the callable workflow reject any model
   other than `deepseek/deepseek-chat`. When you add models later, you'll update the
   `case` allow-list. Prefer that, or would you rather not validate and accept any
   string?

6. **Should the example caller be committed?** I created both the callable workflow
   and an example caller to mirror the existing pair. If you only wanted the
   callable workflow, the example file can be removed.
