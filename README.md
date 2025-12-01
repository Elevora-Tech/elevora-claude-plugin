# Elevora Claude Plugin

Developer productivity tools for Claude Code - catch up on your work context, track team activity, and automate BMAD epics.

## Installation

```bash
# Add the marketplace
/plugin marketplace add Elevora-Tech/elevora-claude-plugin

# Install the plugin
/plugin install elevora-plugin
```

## Commands

### `/catchup`

Catch up on where you left off since your last session. Analyzes your git timeline, work docs, and Jira context to show what changed and suggest next steps.

**Features:**
- Analyzes recent git commits and extracts business-relevant changes
- Detects page URLs from Next.js app routes
- Integrates with Jira (if Atlassian MCP configured)
- Suggests logical next steps based on work patterns
- Filters out styling/refactoring noise to focus on features

**Usage:**
```
/catchup
```

### `/team-pulse`

See your team's pulse - track developer activity across branches and PRs. Shows recent commits, PR status, and categorizes work.

**Features:**
- Fetches all remote branches to include team activity
- Categorizes commits: features, bugs, refactoring, docs
- Shows PR status (requires GitHub CLI)
- Highlights missing PRs and activity gaps
- Configurable time range

**Usage:**
```
/team-pulse
/team-pulse last 3 days
/team-pulse this week
```

### `/complete-epic [epic-number]`

Complete an entire BMAD epic autonomously. Runs through all stories from current state to done.

> **Note:** Requires [BMAD](https://github.com/bmad-code-org/BMAD-METHOD) to be installed in your project.

**Features:**
- Supports BMAD v4 and v6
- Autonomous execution mode
- Test failure recovery (up to 5 attempts)
- Automatic code review handling
- Progress tracking and completion summary

**Usage:**
```
/complete-epic 9
```

## Skills

### `team-pulse`

Model-invoked skill for tracking developer activity. Claude will automatically use this when you ask questions like:
- "What have developers been working on?"
- "Show me what was done in the last 3 days"
- "Who's been working on features this week?"

## Configuration

### Catchup Configuration

Create `.claude/catchup.config.json` in your project root to customize:

```json
{
  "jira": {
    "enabled": true,
    "fetchEpics": true,
    "parseTicketFromBranch": true
  },
  "workDocs": {
    "patterns": ["*_REMAINING_WORK.md", "*WORK_LOG.md"],
    "exclude": ["node_modules/**", "dist/**"]
  },
  "git": {
    "commitLimit": 10,
    "excludePaths": ["package-lock.json", "yarn.lock"]
  },
  "output": {
    "verbosity": "detailed"
  }
}
```

## Requirements

- Claude Code CLI
- Git repository
- GitHub CLI (`gh`) for PR status in team-pulse (optional)
- Atlassian MCP for Jira integration in catchup (optional)
- BMAD installation for complete-epic command

## License

MIT
