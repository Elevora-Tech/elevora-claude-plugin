# Team Pulse - Developer Activity Tracker

A Claude Code skill to track developer activity across any git repository.

## Quick Start

In any git repository, run:

```
/team-pulse
```

Or with natural language:
- "What have developers been working on?"
- "Show me what was done in the last 3 days"
- "Check recent developer activity"
- "Who's been working on features this week?"

## What It Does

1. **Fetches all remote branches** (including other developers' work)
2. **Analyzes commits** from the specified time range (default: 7 days)
3. **Categorizes work**:
   - Features (new functionality)
   - Bug Fixes
   - Refactoring
   - Documentation
   - Configuration/Build
   - Styling (minimized)
4. **Shows PR status** (if GitHub CLI installed)
5. **Flags issues** (missing PRs, low activity)

## Time Range Examples

- "last 3 days" → 3 days
- "last week" → 7 days
- "past 2 weeks" → 14 days
- "yesterday" → 1 day
- No specification → defaults to 7 days

## Output Includes

- Total commits per developer
- Work breakdown by category
- Branch names and PR status
- Line change statistics
- Daily activity breakdown
- Recommendations (e.g., missing PRs)

## Requirements

- Git repository
- GitHub CLI (`gh`) for PR status (optional but recommended)

## Install GitHub CLI

```bash
brew install gh
gh auth login
```

## Use Cases

- **Daily standups**: "What did the team work on yesterday?"
- **Weekly reviews**: "Show developer activity for the last week"
- **Sprint planning**: "What features were completed in the last 2 weeks?"
- **Code review prep**: "Who has open branches without PRs?"

## Filters Out

- Pure styling changes (unless >100 lines)
- Whitespace/formatting only
- Auto-generated files
- Merge commits (except PR merges)
