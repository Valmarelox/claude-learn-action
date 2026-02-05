You are analyzing this repository to identify patterns where Claude made mistakes,
so you can propose rule improvements to prevent future issues.

## PHASE 1: Data Gathering

### Git History Analysis
Run: `git log --oneline --stat --no-merges --since="__DAYS_TO_ANALYZE__ days ago"`

Look for commits that suggest corrections:
- Message patterns: "fix", "revert", "correct", "cleanup", "remove unnecessary", "actually", "oops", "typo"
- Files modified multiple times in short succession
- Small commits following large Claude-generated commits

For each suspicious commit, run `git show <hash>` to see the actual diff.
Focus on WHAT was wrong in the code, not just the commit message.

<!-- BEGIN:PR_ANALYSIS -->
### PR Review Comments (if available)
Run: `gh pr list --state merged --limit 20 --json number,title,mergedAt`

For recent merged PRs, fetch review comments:
`gh pr view <number> --comments --json comments,reviews`

Look for:
- Review feedback explaining why code was wrong
- Requested changes patterns
- Common reviewer complaints

If `gh` fails or no PRs exist, continue with git-only analysis.
<!-- END:PR_ANALYSIS -->

## PHASE 2: Pattern Recognition

Categorize issues found. Start with these categories, but add new ones if you discover patterns that do not fit:

- **style**: Code formatting, naming conventions, idioms
- **architecture**: Wrong patterns, poor structure, coupling issues
- **overengineering**: Unnecessary abstractions, premature optimization
- **underengineering**: Missing error handling, edge cases, validation
- **security**: Vulnerabilities, unsafe practices
- **testing**: Missing tests, poor test quality
- **dependencies**: Wrong imports, version issues

For each pattern, note:
- Frequency (how many times it occurred)
- Severity (how much effort to fix)
- Concrete example from the diffs

## PHASE 3: Rule Proposal

Read existing rules in __RULES_FILE__ and __RULES_PATH__/.

For each significant pattern (occurred 2+ times OR high severity):
1. Check if an existing rule already covers it
2. If not, draft a new rule with:
   - Clear, actionable directive
   - Brief code example (3-5 lines max showing wrong vs right)
   - Category tag

## PHASE 4: Output

If patterns found worth documenting:
- Create/update rule files
- Create a PR with:
  - Title: "chore: update Claude rules based on recent patterns"
  - Body listing patterns found and rules added

If no significant patterns (all one-offs or already covered):
- Exit cleanly without creating anything
- Optionally log what was analyzed

## CONSTRAINTS
- Rule files must stay under __MAX_RULE_LINES__ lines
- Prioritize recurring patterns over one-offs
- Be specific - vague rules do not help
- Do not duplicate existing rules
