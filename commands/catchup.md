---
description: Catch up on where you left off - analyzes your git timeline, work docs, and issue tracker context to show what changed since your last session and suggest next steps
allowed-tools: Bash(git:*), Read(/**), Glob(**), Write(.claude/catchup.config.json), mcp__atlassian__*, linear_*, mcp__github__*
---

# Catchup - Get Back Up to Speed

Analyze the current work state and help me understand where I left off and what to do next.

## Step 1: Load Configuration

Check if `.claude/catchup.config.json` exists in the repository root. If it exists, load the configuration. Otherwise, use these defaults:

```json
{
  "issueTracker": {
    "provider": null,
    "fetchParent": true,
    "parseKeyFromBranch": true
  },
  "workDocs": {
    "patterns": ["*_REMAINING_WORK.md", "*WORK_LOG.md", ".claude/sessions/*.md"],
    "exclude": ["node_modules/**", "dist/**", ".git/**"]
  },
  "git": {
    "commitLimit": 10,
    "includeDiff": true,
    "excludePaths": ["package-lock.json", "yarn.lock", "pnpm-lock.yaml"]
  },
  "output": {
    "showTimeline": true,
    "showNextSteps": true,
    "showTodoList": true,
    "verbosity": "detailed"
  }
}
```

### Backwards Compatibility
If the config has the legacy `jira` key instead of `issueTracker`:
- If `jira.enabled: true` → treat as `issueTracker.provider: "jira"`
- If `jira.enabled: false` → treat as `issueTracker.provider: "none"`
- Map `jira.fetchEpics` → `issueTracker.fetchParent`
- Map `jira.parseTicketFromBranch` → `issueTracker.parseKeyFromBranch`

## Step 1.5: Determine Issue Tracker Provider

### Check Configuration
1. If `issueTracker.provider` exists in config → use that provider
2. If legacy `jira.enabled: true` → use "jira" provider
3. If `issueTracker.provider` is null or missing → **ASK THE USER**

### First-Run Prompt (if no provider configured)
If no issue tracker provider is configured, ask the user:

**"Which issue tracker do you use for this project?"**
- **Jira** - Uses Atlassian MCP (`mcp__atlassian__*` tools)
- **Linear** - Uses Linear MCP (`linear_*` tools)
- **GitHub Issues** - Uses GitHub MCP (`mcp__github__*` tools)
- **None** - Skip issue tracking, use git-only analysis

After user responds, save their selection to `.claude/catchup.config.json`:
```json
{
  "issueTracker": {
    "provider": "<user-selection>",
    "fetchParent": true,
    "parseKeyFromBranch": true
  }
}
```

Confirm: "Saved: Using {provider} for issue tracking. You can change this in `.claude/catchup.config.json`."

Then continue with the selected provider.

## Step 2: Extract Context

### Branch & Issue Key Information
1. Get current branch name: `git branch --show-current`
2. If `parseKeyFromBranch` is enabled, extract issue key from branch name based on provider:
   - **Jira/Linear**: Pattern `/([A-Z]+-\d+)/` → e.g., `PROJ-123`, `ENG-456`
   - **GitHub**: Pattern `/#?(\d+)/` or `/issue[/-]?(\d+)/i` → e.g., `#123`, `issue-456`
3. Get git status to understand current state

### Work Documents
1. Search for work documentation files matching the patterns from config
2. Read any found work docs to understand the feature context

### Git Timeline
1. Get recent commits: `git log -n <commitLimit> --pretty=format:'%h|%ai|%s' --date=iso`
2. For each commit, analyze:
   - Timestamp (convert to configured timezone if specified)
   - Message
   - Files changed: `git show --name-only --pretty=format:'' <commit-hash>`
3. Build chronological timeline of work progression

### Active Todos
1. Check if there are any active Claude Code todos
2. Note completed vs pending items

## Step 3: Fetch Issue Details

If provider is "none", skip this step entirely and proceed to Step 4.

If an issue key was extracted from the branch name, fetch issue details based on the configured provider:

### Jira Provider
1. Use `mcp__atlassian__getAccessibleAtlassianResources` to get cloudId
2. Use `mcp__atlassian__search` with the issue key to find the issue
3. Use `mcp__atlassian__fetch` to get full issue details
4. If `fetchParent` is enabled and issue has a parent epic, fetch that too
5. Extract:
   - Issue title and description
   - Current status
   - Acceptance criteria (if available)
   - Epic/parent context (if applicable)
   - URL: `https://{cloudName}.atlassian.net/browse/{key}`

### Linear Provider
1. Use `linear_search_issues` with the issue key (e.g., `ENG-123`)
2. Extract:
   - Issue title and description
   - Current status (Backlog, Todo, In Progress, In Review, Done, etc.)
   - Labels and priority
   - Parent issue context (if applicable)
   - URL: `https://linear.app/{team}/issue/{key}`

### GitHub Issues Provider
1. Determine repo owner and name from git remote: `git remote get-url origin`
2. Use `mcp__github__get_issue` with owner, repo, and issue number
3. Extract:
   - Issue title and body
   - State (open/closed) and labels
   - Milestone (if any)
   - URL: `https://github.com/{owner}/{repo}/issues/{number}`

### Status Normalization
Map provider-specific statuses to categories for consistent output:

| Category | Jira | Linear | GitHub |
|----------|------|--------|--------|
| `todo` | To Do, Open, Backlog | Backlog, Todo, Triage | open (no in-progress label) |
| `in_progress` | In Progress, In Review, In QA | In Progress, In Review | open + "in-progress" label |
| `done` | Done, Closed, Resolved | Done, Completed, Canceled | closed |
| `blocked` | Blocked, On Hold | Blocked | open + "blocked" label |

### Error Handling for Issue Fetching
If MCP tools fail (not configured, not authenticated, or rate limited):
- Log: "Issue tracker ({provider}) not available - continuing with git-only analysis"
- Do NOT fail the command - continue to Step 4
- Note in output that issue context was unavailable

## Step 4: Detect Page URLs and Navigation

Analyze changed files to determine page locations:
1. **Next.js App Routes**: Parse `app/` directory structure
   - Example: `app/support/dashboard/tickets/page.tsx` → `/support/dashboard/tickets`
   - Example: `app/red-alert/(pages)/tickets/page.tsx` → `/red-alert/tickets`
2. **Component Locations**: Identify related component directories
   - Example: `components/app/SupportDashboard/` → Support Dashboard feature
3. **Navigation Path**: Infer navigation from file structure and app knowledge
   - Look for parent pages, layout files, navigation components
4. **Group by Feature Area**: If changes span multiple pages, list all relevant URLs

## Step 5: Analyze Work Progression (Business-Focused)

**IMPORTANT: Filter out technical/styling details and focus on business value**

### Commit Filtering Rules:
**SKIP these commits entirely:**
- Messages containing: "Refactor", "Update styles", "Enhance", "Improve readability", "Improve responsiveness", "Polish", "Visual consistency", "Better styling", "UI improvements"
- Pure styling changes that don't add business functionality
- Code organization changes that don't affect features

**KEEP and highlight:**
- Messages with: "Add", "Create", "Build", "Implement", "Fix [bug]", "Enable"
- New features or user-facing capabilities
- Business logic changes
- Bug fixes that affect functionality

**GROUP related changes:**
- If 3+ consecutive commits are styling/refactoring the same feature, group them as one business change
- Example: "Refactor card", "Update badge styles", "Improve layout" → "Built/Updated ticket card display"

### Analysis Focus:
1. What business features/capabilities were added or changed
2. The chronological sequence from a product perspective
3. The most recent user-facing change

## Step 6: Generate Summary

Provide a clear, actionable summary:

### Current Work
- **Issue**: [KEY] Title (or "No issue tracker configured" if provider is "none")
- **Status**: Status name (normalized category)
- **Provider**: jira/linear/github/none
- **Branch**: [branch name]
- **Link**: [Direct URL to issue if available]

### Page/Feature Location
**Show URLs and navigation for pages being worked on:**
- **URL**: [Auto-detected URL from app/ directory, e.g., `/support/dashboard/tickets`]
- **Navigation**: [How to get there, e.g., "Dashboard → Support → Tickets tab"]
- **Components**: [Key directories, e.g., `web/app/support/dashboard/tickets/`, `web/components/app/SupportDashboard/`]

**If multiple pages/features:**
List each with its URL and navigation path.

**URL Detection Logic:**
- Parse `app/` directory: `app/support/dashboard/tickets/page.tsx` → `/support/dashboard/tickets`
- Handle route groups: `app/red-alert/(pages)/tickets/page.tsx` → `/red-alert/tickets`
- For components without page files, identify the feature area from directory structure

### Context
[Brief summary of what this feature/work is about from issue description or work docs]

### Work Timeline (Business-Focused)

**IMPORTANT FORMATTING RULES:**
- **Only include commits that added/changed business functionality**
- **Skip all styling/refactoring commits** (see filtering rules in Step 5)
- **Group related minor changes** into one business-level entry
- **Focus on WHAT was built, not HOW it looks**

Present business-relevant commits chronologically:
- Date/time (in configured timezone)
- Business feature or capability added/changed
- User-facing impact (NOT technical implementation details)

**Example format:**
```
Nov 11, 5:57 PM - Built manual ticket creation form
  → Added form with modal interface for creating support tickets
  → Included US state selection for ticket context

Nov 11, 6:43 AM - Added ticket type selection
  → Implemented dropdown for standardized ticket types
```

**What NOT to include:**
```
❌ Nov 11, 4:43 PM - Refactored ServiceTicketCard layout for improved readability
❌ Nov 11, 5:12 PM - Updated support ticket status colors for visual clarity
❌ Nov 11, 8:58 AM - Introduced shared status configuration
```

**Instead, group these as:**
```
✅ Nov 11 - Built ticket status system
  → Added status badges and filtering (open, pending, resolved, done)
```

### Last Step Completed
[Identify the most recent meaningful BUSINESS change - what feature/capability was added or fixed, NOT styling/refactoring]

### Suggested Next Step
[Based on work docs, todos, issue acceptance criteria, and the progression pattern, suggest the logical next step]

### Additional Context
- Link to work docs if found
- Active todos if any
- Issue link (direct URL to the issue in the configured provider)
- Untracked files if relevant to the feature

## Output Guidelines

### Business-Focused Output:
- **Filter aggressively**: Skip commits about styling, refactoring, visual improvements unless they enable new functionality
- **Focus on features**: What can users do now that they couldn't before?
- **Use business language**: "Built ticket creation form" not "Refactored form component architecture"
- **Group minor changes**: Combine styling/polish commits into one business-level summary
- **Show page locations**: Always include URLs and how to navigate to the pages being worked on

### Formatting:
- **Do NOT list individual files changed** - focus on what was accomplished from a product perspective
- Use clear section headers for scannability
- Keep timeline entries concise but meaningful
- Infer business value from file paths (e.g., new pages = new features, hooks = new capabilities)
- Be specific about the suggested next step based on patterns observed
- If verbosity is "brief", condense sections significantly and show only top 3 business changes
- If verbosity is "verbose", include more context about each business change

### Page URL Guidelines:
- Always show the primary page/feature URL at the top
- Include navigation instructions (how to get there in the app)
- List component directories for developer context
- If working across multiple pages, list all relevant URLs

## Error Handling

- If no git repository: "Not in a git repository"
- If no commits: "No commits found in current branch"
- If issue tracker MCP not available: Continue without issue data, note "Issue tracker ({provider}) not available" in output
- If issue key not found in tracker: Note "Issue not found", continue with git analysis
- If work docs not found: Continue with git analysis only
- If config file is malformed: Use defaults and warn user
- If first-run and user selects a provider but MCP fails: Warn user to configure the MCP, offer to try again or use "none"
