---
description: Catch up on where you left off - analyzes your git timeline, work docs, and Jira context to show what changed since your last session and suggest next steps
allowed-tools: Bash(git:*), Read(/**), Glob(**), mcp__atlassian__*
---

# Catchup - Get Back Up to Speed

Analyze the current work state and help me understand where I left off and what to do next.

## Step 1: Load Configuration

Check if `.claude/catchup.config.json` exists in the repository root. If it exists, load the configuration. Otherwise, use these defaults:

```json
{
  "jira": {
    "enabled": true,
    "fetchEpics": true,
    "parseTicketFromBranch": true
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

## Step 2: Extract Context

### Branch & Ticket Information
1. Get current branch name: `git branch --show-current`
2. If `parseTicketFromBranch` is enabled, extract ticket ID (e.g., PROJ-12135 from branch name)
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

## Step 3: Fetch Jira Details (if enabled)

If Jira integration is enabled and a ticket ID was found:
1. Use `mcp__atlassian__getAccessibleAtlassianResources` to get cloudId
2. Use `mcp__atlassian__search` with the ticket ID to find the issue
3. Use `mcp__atlassian__fetch` to get full ticket details
4. If `fetchEpics` is enabled and ticket has a parent epic, fetch that too
5. Extract:
   - Ticket title and description
   - Current status
   - Acceptance criteria (if available)
   - Epic context (if applicable)

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
- **Ticket**: [ID and title from Jira]
- **Branch**: [branch name]
- **Status**: [Jira status if available]

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
[Brief summary of what this feature/work is about from Jira description or work docs]

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
[Based on work docs, todos, Jira acceptance criteria, and the progression pattern, suggest the logical next step]

### Additional Context
- Link to work docs if found
- Active todos if any
- Jira link (using customContext.ticketUrlPattern if configured)
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
- If Jira enabled but credentials missing: Continue without Jira data, note it in output
- If work docs not found: Continue with git analysis only
- If config file is malformed: Use defaults and warn user
