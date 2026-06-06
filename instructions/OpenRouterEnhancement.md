# OpenRouter enablement for RuFlo
Create a new callable workflow that is patterned after ruflo-hivemind.yml and ruflo-hivemind-example.yml that overrides use anthropic LLMs and uses DeepSeek. My intent is to execute RuFlo using Claude Code, but using DeepSeek LLMs behind the scenes. Bash-level instructions for the override are provided in this document in a section below.

Detailed requirements are:
- Assume that the OpenRouter API key is provided by a GitHub secret
- Accept an input for the model. Default to DeepSeek and make it the only option, but I will add more later.
- Like the workflow ruflo-hivemind.yml, note output from the various RuFlo steps in the workflow execution summary.
- Like the workflow ruflo-hivemind.yml, accept an instruction and RuFlo version as input. 
- Document any questions you have for me in file notes/openrouter_questions_v1.md
- Document your thinking in file notes/openrouter_questions_v1.md

## DeepSeek override instructions

```
### Step 1: Define the Environment Variables

OpenRouter exposes an Anthropic-compatible gateway at `[https://openrouter.ai/api](https://openrouter.ai/api)`. Run the following commands in your terminal to set up the mapping. 

Replace `<your_openrouter_api_key>` with your actual credential:

```bash
# Point Claude Code to OpenRouter's gateway layer
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export OPENROUTER_API_KEY="<your_openrouter_api_key>"

# Crucial: Override the standard auth token with your OpenRouter key
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"

# Crucial: Explicitly blank out the native key to force the client to use the base URL
export ANTHROPIC_API_KEY=""

# Map the primary execution models to DeepSeek Pro (Chat/Reasoning v4)
export ANTHROPIC_MODEL="deepseek/deepseek-chat"
export ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek/deepseek-chat"
export ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek/deepseek-chat"

# Map the secondary/fast execution tiers to DeepSeek Flash
export ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek/deepseek-flash"
export CLAUDE_CODE_SUBAGENT_MODEL="deepseek/deepseek-flash"

# Instruct Claude Code to ignore organizational check barriers for OpenRouter's fast mode compatibility
export CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK=1
```