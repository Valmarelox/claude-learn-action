# Claude Self-Learning Review Action

A GitHub Action that analyzes your repository's recent commits to identify patterns where Claude made mistakes, then automatically proposes rule improvements via pull requests.

## How It Works

1. Analyzes git history for fix/revert commits that suggest Claude errors
2. Reads existing rules from `CLAUDE.md` and `.claude/rules/`
3. Identifies gaps where rules could have prevented issues
4. Creates a PR with specific rule improvements (if patterns found)

## Usage

### Basic Setup

Create `.github/workflows/claude-self-learning.yml`:

```yaml
name: Claude Self-Learning Review

on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9:00 AM UTC
  workflow_dispatch:
    inputs:
      prompt:
        description: 'Custom prompt for Claude'
        required: false
        type: string

jobs:
  self-learning-review:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      id-token: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: your-username/claude-learn-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          prompt: ${{ github.event.inputs.prompt }}
```

### With Custom Configuration

```yaml
- uses: your-username/claude-learn-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    days_to_analyze: '14'
    rules_path: 'docs/rules'
    rules_file: 'CONTRIBUTING.md'
    max_rule_lines: '150'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `claude_code_oauth_token` | Claude Code OAuth token | Yes* | - |
| `anthropic_api_key` | Anthropic API key (alternative auth) | Yes* | - |
| `days_to_analyze` | Days of git history to analyze | No | `7` |
| `prompt` | Custom prompt (overrides default) | No | - |
| `rules_path` | Path to rules directory | No | `.claude/rules` |
| `rules_file` | Path to main rules file | No | `CLAUDE.md` |
| `max_rule_lines` | Maximum lines per rule file | No | `200` |
| `allowed_tools` | Tools Claude can use | No | `Bash(git *),Edit,Write,Read,Glob,Grep` |
| `model` | Claude model to use | No | - |

*One of `claude_code_oauth_token` or `anthropic_api_key` is required.

## Outputs

| Output | Description |
|--------|-------------|
| `pr_number` | Pull request number (if created) |
| `pr_url` | Pull request URL (if created) |

## Requirements

- Repository must have git history (use `fetch-depth: 0` in checkout)
- Workflow needs `contents: write` and `pull-requests: write` permissions

## License

MIT
