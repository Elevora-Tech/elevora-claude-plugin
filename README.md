# Elevora Claude Plugin

Developer productivity tools for Claude Code - catch up on your work context, track team activity, automate BMAD epics, and audit production readiness.

## Installation

```bash
# Add the marketplace
/plugin marketplace add Elevora-Tech/elevora-claude-plugin

# Install the plugin
/plugin install elevora-plugin@elevora-marketplace
```

## Commands

### `/catchup`

Catch up on where you left off since your last session. Analyzes your git timeline, work docs, and issue tracker context to show what changed and suggest next steps.

**Features:**
- Analyzes recent git commits and extracts business-relevant changes
- Detects page URLs from Next.js app routes
- Integrates with **Jira**, **Linear**, or **GitHub Issues** (asks on first run)
- Suggests logical next steps based on work patterns
- Filters out styling/refactoring noise to focus on features

**Issue Tracker Support:**

| Provider | MCP Required | Key Pattern |
|----------|-------------|-------------|
| Jira | Atlassian MCP (`mcp__atlassian__*`) | `PROJ-123` |
| Linear | Linear MCP (`linear_*`) | `ENG-456` |
| GitHub Issues | GitHub MCP (`mcp__github__*`) | `#789` |

On first run, you'll be asked which issue tracker you use. Your choice is saved to `.claude/catchup.config.json`.

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

### `/eda-architect`

Inngest architecture expert — analyzes your codebase, plans workflow decomposition, recommends flow control patterns, and designs migration strategies from legacy queues to Inngest.

**Features:**
- Scans for existing Inngest functions and legacy queue systems (BullMQ, Celery, SQS, RabbitMQ, Temporal, Sidekiq)
- Health checks Inngest functions for step hygiene, flow control fitness, and edge case violations
- Recommends function decomposition (single vs. multiple coordinated functions)
- Suggests flow control config (concurrency, throttle, rateLimit, debounce, idempotency, priority, batch)
- Plans migrations from legacy queues with concept mappings and phased approach
- Read-only analysis — never writes or modifies files

**Usage:**
```
/eda-architect
/eda-architect focus on the payments service
```

### `/ship-check`

Production readiness audit — spawns 7 specialist agents to evaluate whether your web app is ready for real users to pay for and succeed with.

**The 7 Specialists:**

| Agent | Focus |
|-------|-------|
| Security Auditor | OWASP Top 10, auth, secrets, headers, input validation |
| Feature Completeness Analyst | Routes, CRUD, error/loading/empty states, TODOs |
| UX & Accessibility Inspector | WCAG 2.1 AA, keyboard nav, responsive, mobile, forms |
| Infrastructure & DevOps Reviewer | CI/CD, logging, monitoring, env config, deployment |
| Performance Analyst | Bundle size, images, caching, DB queries, Core Web Vitals |
| Business & Launch Reviewer | SEO, analytics, legal, payments, email, social proof |
| Product Experience Strategist | UVP, onboarding, first-run, differentiation, retention |

**Verdict Scale:** Not Production Ready → MVP - Needs Work → MVP - Ready for Beta → Production Ready → Launch Ready - Ship It

**Usage:**
```
/ship-check
/ship-check security
/ship-check product
```

## Skills

### `team-pulse`

Model-invoked skill for tracking developer activity. Claude will automatically use this when you ask questions like:
- "What have developers been working on?"
- "Show me what was done in the last 3 days"
- "Who's been working on features this week?"

### `eda-architect`

Model-invoked skill for Inngest architecture analysis. Claude will automatically use this when you ask questions like:
- "Should this be one Inngest function or multiple?"
- "How should I structure this workflow in Inngest?"
- "Migrate this BullMQ code to Inngest"
- "Review my Inngest functions for issues"
- "What flow control should I use here?"

### `ship-check`

Model-invoked skill for production readiness auditing. Claude will automatically use this when you ask questions like:
- "Is this app ready to ship?"
- "Can we launch this?"
- "What's blocking production readiness?"
- "Do a pre-launch check"
- "Is this ready for users?"

## Configuration

### Catchup Configuration

Create `.claude/catchup.config.json` in your project root to customize:

```json
{
  "issueTracker": {
    "provider": "jira",
    "fetchParent": true,
    "parseKeyFromBranch": true
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

**Issue Tracker Options:**
- `provider`: `"jira"` | `"linear"` | `"github"` | `"none"`
- `fetchParent`: Fetch parent issues (epics in Jira, parent issues in Linear)
- `parseKeyFromBranch`: Extract issue keys from git branch names

**Changing Your Provider:**
Edit the `issueTracker.provider` value in your config file, or delete the config to be prompted again.

## Requirements

- Claude Code CLI
- Git repository
- GitHub CLI (`gh`) for PR status in team-pulse (optional)
- BMAD installation for complete-epic command

**Issue Tracker MCPs (optional - for `/catchup`):**
- [Atlassian MCP](https://github.com/atlassian/mcp-server-atlassian) for Jira
- [Linear MCP](https://mcp.linear.app) for Linear (official)
- [GitHub MCP](https://github.com/github/github-mcp-server) for GitHub Issues

## License

MIT
