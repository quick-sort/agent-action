# agent-action

[![Test Action](https://github.com/quick-sort/agent-action/actions/workflows/test.yml/badge.svg)](https://github.com/quick-sort/agent-action/actions/workflows/test.yml)

A reusable GitHub Action that installs and configures [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI with a custom LLM endpoint. Designed for non-interactive CI/CD usage.

## Quick Start

1. Add secrets to your GitHub repo (Settings → Secrets and variables → Actions), or to an environment (Settings → Environments):
   - `LLM_API_KEY` — your API key
   - `LLM_BASE_URL` — your API endpoint URL

2. Add to your workflow:

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: quick-sort/agent-action@main
    with:
      llm_api_key: ${{ secrets.LLM_API_KEY }}
      llm_base_url: ${{ secrets.LLM_BASE_URL }}

  - run: agent-do "fix lint errors in src/"
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `llm_api_key` | ✅ | | LLM API key |
| `llm_base_url` | ✅ | | LLM API base URL |
| `fast_model` | | `claude-haiku` | Fast model name |
| `default_model` | | `claude-sonnet` | Default model name |
| `smart_model` | | `claude-opus` | Smart model name |
| `node_version` | | `20` | Node.js version |
| `claude_code_version` | | *(latest)* | Pin Claude Code package version |
| `permission_mode` | | `auto` | Permission mode: `default`, `acceptEdits`, `plan`, `auto`, `bypassPermissions` |
| `tavily_api_key` | | | Tavily API key for web search MCP |

## Outputs

| Output | Description |
|---|---|
| `claude_code_version` | Installed Claude Code version |

## Usage

### Basic

```yaml
name: Claude Code Task
on:
  workflow_dispatch:
    inputs:
      prompt:
        description: 'Prompt for Claude'
        required: true

jobs:
  run:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - uses: quick-sort/agent-action@main
        with:
          llm_api_key: ${{ secrets.LLM_API_KEY }}
          llm_base_url: ${{ secrets.LLM_BASE_URL }}

      - run: agent-do "${{ github.event.inputs.prompt }}"
```

### Custom Models

```yaml
- uses: quick-sort/agent-action@main
  with:
    llm_api_key: ${{ secrets.LLM_API_KEY }}
    llm_base_url: ${{ secrets.LLM_BASE_URL }}
    fast_model: my-haiku
    default_model: my-sonnet
    smart_model: my-opus
```

### Auto-fix and Commit

```yaml
- uses: actions/checkout@v4

- uses: quick-sort/agent-action@main
  with:
    llm_api_key: ${{ secrets.LLM_API_KEY }}
    llm_base_url: ${{ secrets.LLM_BASE_URL }}

- run: agent-do "review and fix issues in this repo"

- name: Commit changes
  run: git diff --quiet || (git add -A && git commit -m "fix: auto-fix by claude" && git push)
```

### Use claude Directly

`agent-do` is a convenience wrapper. You can also call `claude` directly:

```yaml
- run: claude -p --permission-mode auto --output-format text "your prompt"
```

## How It Works

1. Installs Node.js and Claude Code CLI globally
2. Sets environment variables (`ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL`, model configs, etc.) via `$GITHUB_ENV`
3. Copies a default `.claude.json` (skips project onboarding) if one doesn't exist
4. Creates an `agent-do` wrapper script that calls `claude -p --permission-mode <mode> --output-format text`

## License

MIT
