---
description: See your team's pulse - track developer activity across branches and PRs. Shows recent commits, PR status, and categorizes work (features, bugs, refactoring). Use with optional time range (e.g., "last 3 days") or defaults to 7 days.
---

Invoke the `team-pulse` skill to analyze developer activity in this repository.

Pass through any time range specification from the user's request (e.g., "last 3 days", "this week", "yesterday").

The skill will:
1. Fetch all remote branches
2. Analyze commits from the specified time period
3. Categorize work (features, bugs, refactoring, etc.)
4. Show PR status and activity breakdown
5. Flag any missing PRs or issues
