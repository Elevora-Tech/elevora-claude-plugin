---
description: Complete an entire BMAD epic autonomously (v4 & v6 compatible)
argument-hint: '[epic-number]'
---

> **Requires BMAD**: This command works with the BMAD (Build-Measure-Analyze-Decide) methodology.
> If you don't have BMAD installed, visit: https://github.com/bmad-code-org/BMAD-METHOD
> to learn more and get started.

# Complete Epic - Autonomous BMAD Epic Execution

Execute all workflows to complete epic **$1** from current state through all stories to done.

**EXECUTION MODE: #yolo** - Run fully autonomously without user confirmations.

---

## CRITICAL EXECUTION RULES

1. **#yolo mode active** - Skip all confirmations, simulate expert user decisions
2. **Never pause for approval** - Continue automatically between workflows
3. **Never ask "should we continue?"** - The answer is always yes
4. **Test failure recovery** - Attempt up to 5 fix cycles before halting
5. **Code review changes** - Automatically loop back to dev-story without asking
6. **Complete ALL stories** - Don't stop until every story in the epic is done

---

## PHASE 0: Version Detection & Configuration

### Step 0.1: Detect BMAD Version

Check which BMAD version is installed:

```
IF file exists: {project-root}/.bmad/_cfg/manifest.yaml
  → BMAD_VERSION = 6
  → BMAD_FOLDER = ".bmad"
  → CONFIG_PATH = ".bmad/bmm/config.yaml"
  → Read config to get: sprint_artifacts, output_folder
  → STATUS_FILE = "{sprint_artifacts}/sprint-status.yaml"

ELSE IF file exists: {project-root}/.bmad-core/install-manifest.yaml
  → BMAD_VERSION = 4
  → BMAD_FOLDER = ".bmad-core"
  → CONFIG_PATH = ".bmad-core/core-config.yaml"
  → Read config to get: devStoryLocation, prd, architecture paths
  → STATUS_FILE = null (v4 tracks status in story files)

ELSE
  → ERROR: "No BMAD installation found. Expected .bmad/ (v6) or .bmad-core/ (v4)"
  → HALT
```

### Step 0.2: Load Epic Information

**For BMAD v6:**
1. Read sprint-status.yaml completely
2. Find epic-$1 entry and get current status (backlog/contexted/done)
3. Find all stories for epic $1 (keys matching pattern: $1-*-*)
4. List each story with its current status

**For BMAD v4:**
1. Scan devStoryLocation for files matching: $1.*.story.md
2. Read each story file to extract status from frontmatter
3. List each story with its current status

**Output epic summary:**
```
EPIC $1 STATUS REPORT
=====================
BMAD Version: {BMAD_VERSION}
Epic Status: {epic_status}
Stories Found: {count}

Story Status Breakdown:
- backlog: {count}
- drafted: {count}
- ready-for-dev: {count}
- in-progress: {count}
- review: {count}
- done: {count}
```

---

## PHASE 0.5: Pre-Flight Check

### Step 0.5.1: Check Required Prerequisites

Scan for required input files:

**For BMAD v6:**
- PRD: Check {output_folder}/*prd*.md OR {output_folder}/*prd*/*.md
- Architecture: Check {output_folder}/*architecture*.md OR {output_folder}/*architecture*/*.md
- Epic Tech Spec: Check {sprint_artifacts}/tech-spec-epic-$1.md

**For BMAD v4:**
- PRD: Check prd.prdFile OR prd.prdShardedLocation from config
- Architecture: Check architecture.architectureFile OR architectureShardedLocation

### Step 0.5.2: Report Missing Prerequisites

```
IF any required files are missing:
  List all missing prerequisites:

  MISSING PREREQUISITES
  =====================
  [ ] PRD document - Required for story context
  [ ] Architecture document - Required for technical decisions
  [ ] Epic tech spec - Required for story implementation (v6 only)

  These can be created by running prerequisite workflows.

  ASK ONCE: "Create missing prerequisites? (y/n)"

  IF user says yes:
    → Run /bmad:bmm:workflows:prd (if PRD missing)
    → Run /bmad:bmm:workflows:architecture (if architecture missing)
    → Continue to Phase 1

  IF user says no:
    → Continue anyway with available context
```

---

## PHASE 1: Epic Context (if needed)

### Step 1.1: Check Epic Context Status

```
IF BMAD_VERSION == 6:
  Read sprint-status.yaml
  Get status for epic-$1

  IF status == "backlog":
    → Epic needs contexting, proceed to Step 1.2
  ELSE IF status == "contexted" OR status == "done":
    → Skip to Phase 2

IF BMAD_VERSION == 4:
  → Skip to Phase 2 (v4 doesn't have epic-tech-context workflow)
```

### Step 1.2: Run Epic Tech Context Workflow (v6 only)

```
Execute: /bmad:bmm:workflows:epic-tech-context

Input: Epic number $1
Expected output: {sprint_artifacts}/tech-spec-epic-$1.md
Expected status change: epic-$1: backlog → contexted

When prompted for options, respond 'y' to enter #yolo mode.
Continue automatically when complete.
```

---

## PHASE 2: Story Pipeline

### Main Loop: Process All Stories

```
WHILE any story in epic $1 has status != "done":

  1. Re-read status file to get current state of all stories
  2. Find first story (by story number) that is NOT done
  3. Route to appropriate sub-phase based on status:

  ┌─────────────────────────────────────────────────────────┐
  │ STATUS: backlog                                         │
  │ → Go to Phase 2.1: Create Story                         │
  ├─────────────────────────────────────────────────────────┤
  │ STATUS: drafted                                         │
  │ → Go to Phase 2.2: Story Context                        │
  ├─────────────────────────────────────────────────────────┤
  │ STATUS: ready-for-dev OR in-progress                    │
  │ → Go to Phase 2.3: Develop Story                        │
  ├─────────────────────────────────────────────────────────┤
  │ STATUS: review                                          │
  │ → Go to Phase 2.4: Code Review                          │
  └─────────────────────────────────────────────────────────┘

  After completing any sub-phase, return to start of loop.
```

### Phase 2.1: Create Story

**For BMAD v6:**
```
Execute: /bmad:bmm:workflows:create-story

When prompted, respond 'y' to enter #yolo mode.
Expected: Creates {story_key}.md file
Expected status change: {story_key}: backlog → drafted
```

**For BMAD v4:**
```
Load task: .bmad-core/tasks/create-next-story.md
Execute the task for epic $1
Expected: Creates $1.{next_num}.story.md file
Expected status in file: Draft
```

### Phase 2.2: Story Context / Mark Ready

**For BMAD v6:**
```
Execute: /bmad:bmm:workflows:story-context

When prompted, respond 'y' to enter #yolo mode.
Expected: Creates {story_key}.context.xml file
Expected status change: {story_key}: drafted → ready-for-dev
```

**For BMAD v4:**
```
v4 does not have story-context workflow.
Update story file status: Draft → Approved
Continue to Phase 2.3.
```

### Phase 2.3: Develop Story

**For BMAD v6:**
```
Execute: /bmad:bmm:workflows:dev-story

CRITICAL DIRECTIVES (from workflow):
- Do NOT stop for "milestones" or "significant progress"
- Continue until ALL tasks/subtasks are checked
- Only Step 6 decides completion

When prompted, respond 'y' to enter #yolo mode.
Expected status change: ready-for-dev → in-progress → review
```

**For BMAD v4:**
```
Load agent: .bmad-core/agents/dev.md
Load story file: {devStoryLocation}/$1.{story_num}.story.md
Execute all tasks in the story to completion.
Update status in file: Approved → In Progress → Done
```

#### Test Failure Recovery (Both Versions)

```
IF tests fail during development:

  attempt_count = 0
  max_attempts = 5

  WHILE tests failing AND attempt_count < max_attempts:
    attempt_count += 1

    Log: "TEST FAILURE - Fix attempt {attempt_count}/5"

    1. Analyze test failure output
    2. Identify root cause
    3. Implement fix
    4. Re-run tests

    IF tests pass:
      → Continue with story development

  END WHILE

  IF attempt_count >= 5 AND tests still failing:
    Log detailed error report:

    HALT: TEST FAILURE - MAX ATTEMPTS EXCEEDED
    ==========================================
    Story: {story_key}
    Failing tests: {list of failing tests}
    Last error: {error message}
    Fix attempts: 5/5 exhausted

    Suggested actions:
    1. Review test expectations vs implementation
    2. Check for environment issues
    3. Consider if test needs updating

    → HALT execution and wait for user intervention
```

### Phase 2.4: Code Review

**For BMAD v6:**
```
Execute: /bmad:bmm:workflows:code-review

When prompted, respond 'y' to enter #yolo mode.

Expected outcomes:
- APPROVED → status changes to "done", continue to next story
- CHANGES REQUESTED → status changes to "in-progress"
- BLOCKED → status stays "review"
```

**For BMAD v4:**
```
Load task: .bmad-core/tasks/qa-gate.md OR load agent: .bmad-core/agents/qa.md
Execute review on the story.
If approved, update status: In Progress → Done
```

#### Code Review Loop (Both Versions)

```
IF code review outcome == "CHANGES REQUESTED":

  Log: "CODE REVIEW - Changes requested, returning to dev-story"

  DO NOT ask user for confirmation.
  Automatically return to Phase 2.3 (Develop Story).

  The story status will be "in-progress" which routes to dev-story.
  dev-story will see the review findings and address them.

  After addressing findings, story returns to "review" status.
  Loop continues until APPROVED or BLOCKED.

IF code review outcome == "BLOCKED":

  Log: "CODE REVIEW - Blocked, requires manual intervention"

  HALT: STORY BLOCKED BY CODE REVIEW
  ==================================
  Story: {story_key}
  Blocking reason: {reason from review}

  This story cannot proceed without manual intervention.
  → Skip to next story OR halt if no other stories to process
```

---

## PHASE 3: Completion

### Step 3.1: Verify All Stories Complete

```
Re-read status file one final time.
Count stories by status for epic $1.

IF all stories have status == "done":
  → Proceed to completion summary

ELSE:
  Log remaining work:

  INCOMPLETE STORIES
  ==================
  {list stories not in "done" status with their current status}

  IF any stories are blocked:
    → HALT for manual intervention
  ELSE:
    → Return to Phase 2 main loop
```

### Step 3.2: Generate Completion Summary

```
EPIC $1 COMPLETION SUMMARY
==========================
BMAD Version: {BMAD_VERSION}
Total Stories: {count}
Completed: {count}
Skipped: {count} (if any)
Blocked: {count} (if any)

Story Timeline:
{For each story in order}
- {story_key}: {title}
  Started: {timestamp}
  Completed: {timestamp}
  Duration: {calculated}

Total Epic Duration: {calculated}

All story files: {sprint_artifacts}/
Tech spec: {sprint_artifacts}/tech-spec-epic-$1.md

EPIC $1 COMPLETE!
```

### Step 3.3: Optional Retrospective

```
Ask user (only prompt in entire workflow):

"Epic $1 is complete. Run retrospective workflow? (y/n)"

IF yes:
  Execute: /bmad:bmm:workflows:retrospective
  (or .bmad-core equivalent for v4)

IF no:
  Complete command execution.
```

---

## Error Handling

### Configuration Errors
- Missing config file → Use sensible defaults and warn
- Malformed YAML → Report specific parsing error and halt

### Workflow Errors
- Workflow not found → Check both v4 and v6 paths, report if missing
- Workflow fails → Retry once, then report and continue to next story

### Status Tracking Errors
- Cannot update status file → Warn but continue
- Status file corrupt → Attempt repair from story file states

---

## Status Value Reference

| Concept | v6 Value | v4 Value |
|---------|----------|----------|
| Not started | `backlog` | (file doesn't exist) |
| Story created | `drafted` | `Draft` |
| Ready to develop | `ready-for-dev` | `Approved` |
| In development | `in-progress` | `In Progress` |
| Under review | `review` | (N/A in v4) |
| Completed | `done` | `Done` |

---

## Workflow Command Reference

| Phase | v6 Command | v4 Equivalent |
|-------|------------|---------------|
| Epic context | `/bmad:bmm:workflows:epic-tech-context` | (N/A) |
| Create story | `/bmad:bmm:workflows:create-story` | `*create-next-story` |
| Story context | `/bmad:bmm:workflows:story-context` | (N/A) |
| Develop story | `/bmad:bmm:workflows:dev-story` | Load `dev.md` agent |
| Code review | `/bmad:bmm:workflows:code-review` | `*qa-gate` |
| Mark done | `/bmad:bmm:workflows:story-done` | Update story file |

---

## Usage

```bash
# Complete all stories in epic 9
/complete-epic 9

# Start from wherever epic 4 currently is
/complete-epic 4
```

The command will automatically:
1. Detect your BMAD version
2. Find where the epic left off
3. Complete all remaining work
4. Report when finished
