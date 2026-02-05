# Claude Self-Learning Review Action

A GitHub Action that analyzes your repository's recent commits to identify patterns where Claude made mistakes, then automatically proposes rule improvements via pull requests.

## How It Works

1. Analyzes git history for fix/revert commits that suggest Claude errors
2. Optionally reads merged PR review comments for reviewer feedback
3. Reads existing rules from `CLAUDE.md` and `.claude/rules/`
4. Identifies gaps where rules could have prevented issues
5. Creates a PR with specific rule improvements (if patterns found)

## Usage

### Basic Setup

Create `.github/workflows/claude-self-learning.yml`:

```yaml
name: Claude Self-Learning Review

on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9:00 AM UTC
  workflow_dispatch:

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

      - uses: Valmarelox/claude-learn-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

### With Custom Configuration

```yaml
- uses: Valmarelox/claude-learn-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    days_to_analyze: '14'
    rules_path: 'docs/rules'
    rules_file: 'CONTRIBUTING.md'
    max_rule_lines: '150'
    additional_prompt: 'Pay special attention to our SQL query patterns.'
```

### Disabling PR Comment Analysis

PR review comment analysis uses the `gh` CLI which requires `GITHUB_TOKEN` to be available (it is by default in GitHub Actions). To disable it:

```yaml
- uses: Valmarelox/claude-learn-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    analyze_prs: 'false'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `claude_code_oauth_token` | Claude Code OAuth token | Yes* | - |
| `anthropic_api_key` | Anthropic API key (alternative auth) | Yes* | - |
| `days_to_analyze` | Days of git history to analyze | No | `7` |
| `prompt` | Custom prompt (completely overrides default) | No | - |
| `additional_prompt` | Extra context appended to the default prompt | No | - |
| `rules_path` | Path to rules directory | No | `.claude/rules` |
| `rules_file` | Path to main rules file | No | `CLAUDE.md` |
| `max_rule_lines` | Maximum lines per rule file | No | `200` |
| `allowed_tools` | Tools Claude can use | No | `Bash(git *),Edit,Write,Read,Glob,Grep` |
| `model` | Claude model to use | No | - |
| `analyze_prs` | Analyze merged PR review comments | No | `true` |

\*One of `claude_code_oauth_token` or `anthropic_api_key` is required.

**Note on `prompt` vs `additional_prompt`:** `prompt` replaces the entire default analysis prompt. `additional_prompt` appends project-specific context to the default prompt. If both are set, `prompt` takes precedence and `additional_prompt` is ignored.

## Outputs

| Output | Description |
|--------|-------------|
| `pr_number` | Pull request number (if created) |
| `pr_url` | Pull request URL (if created) |

## Requirements

- Repository must have git history (use `fetch-depth: 0` in checkout)
- Workflow needs `contents: write` and `pull-requests: write` permissions
- PR comment analysis (`analyze_prs: true`) requires `GITHUB_TOKEN` with repo read access (available by default in GitHub Actions)

## Prompt Template

The default analysis prompt lives in [`prompt.md`](./prompt.md). It uses a multi-phase approach:

1. **Data Gathering** - git history analysis + optional PR review comments
2. **Pattern Recognition** - categorizes issues by type and severity
3. **Rule Proposal** - drafts rules for recurring patterns
4. **Output** - creates a PR or exits cleanly

To customize behavior without replacing the entire prompt, use `additional_prompt`.

## License

MIT
