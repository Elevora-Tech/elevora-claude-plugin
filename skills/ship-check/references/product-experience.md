# Product Experience Reference Checklist

Detailed checklist for the Product Experience Strategist agent. Evaluate every applicable item against the target codebase.

This checklist addresses the core question: **"Will users understand what this app does, succeed with it quickly, and come back?"**

---

## 1. Unique Value Proposition (UVP)

### Above-the-Fold Clarity
- [ ] Homepage hero text answers "What does this do?" in one sentence
- [ ] Tagline/subtitle answers "Why should I care?" or "How is this different?"
- [ ] CTA is specific (not just "Get Started" — e.g., "Start your free project")
- [ ] Visual (screenshot, demo, animation) shows the actual product
- [ ] Value prop is benefit-focused (not feature-focused)

**Grep patterns:**
```
hero|Hero|tagline|subtitle|headline                 # hero section content
h1|<h1                                              # primary heading
get.?started|sign.?up|try.?free|start.?free         # CTA text
```

### What to Evaluate
- Read the homepage hero section. Can you answer these questions:
  1. What does this product do?
  2. Who is it for?
  3. Why should I choose this over alternatives?
  4. What will I get if I sign up?
- If any answer is unclear, that's a finding.

### Differentiation Signals
- [ ] Unique selling points are explicitly stated (not just generic benefits)
- [ ] Comparison with alternatives (features table, "Why us" section)
- [ ] Specific numbers or outcomes (not vague claims like "faster" or "better")
- [ ] Target audience is clearly defined somewhere on the site

**Grep patterns:**
```
compare|comparison|vs\.|versus|alternative          # comparison content
why.*choose|why.*us|different|unique                # differentiation content
faster|better|easier|simpler                        # vague claims (check for specifics)
```

---

## 2. First-Run Experience

### Post-Signup State
- [ ] Dashboard/home after signup is NOT empty (most critical finding)
- [ ] If dashboard is empty, there's clear guidance on what to do next
- [ ] Sample data or templates help users understand the product
- [ ] Onboarding checklist or progress indicator guides first actions

**What to check:**
1. Find the main authenticated page/dashboard
2. Check what renders when there's no user data
3. Look for empty state components, onboarding flows, or sample data

**Grep patterns:**
```
onboarding|Onboarding|welcome|Welcome               # onboarding flow
checklist|Checklist|get.?started|GetStarted          # progress checklist
sample|template|example|demo.?data                   # sample content
empty.*dashboard|dashboard.*empty                    # empty dashboard handling
first.?time|firstTime|isNew|is.?new                  # first-time user detection
tour|Tour|walkthrough|Walkthrough                    # guided tour
```

### Time to Value
- [ ] User can accomplish the core task within 2 minutes of login
- [ ] No unnecessary setup steps before the user can experience value
- [ ] Most critical action is prominently featured (not buried in navigation)
- [ ] Progressive disclosure — don't overwhelm with all features at once

### Setup Flow
- [ ] Profile completion is optional (don't block value with mandatory setup)
- [ ] Required setup steps are minimal and clearly justified
- [ ] Setup can be completed later (skip option exists)
- [ ] Setup flow has progress indicator

**Grep patterns:**
```
setup|Setup|wizard|Wizard                           # setup flows
step.*\d|Step.*\d|progress                          # multi-step indicators
skip|Skip|later|Later|remind                        # skip/defer options
required.*profile|mandatory.*setup                   # blocking requirements
```

---

## 3. Onboarding Flow

### Guided Experience
- [ ] New users are guided through key features
- [ ] Onboarding is contextual (appears when relevant, not all at once)
- [ ] Onboarding can be dismissed and revisited
- [ ] Onboarding adapts to user role/plan (if applicable)

**Grep patterns:**
```
tooltip|Tooltip|popover|Popover                     # contextual guidance
guided|guide|tutorial                               # guided experience
dismiss|close.*tour|skip.*tour                      # dismissible onboarding
step.*1.*step.*2|wizard                             # sequential steps
```

### Educational Content
- [ ] Feature discovery hints exist (tooltips, callouts for new features)
- [ ] Help text on complex features
- [ ] Documentation linked from within the app
- [ ] Empty states teach (explain what the feature does and how to use it)

**Grep patterns:**
```
help.*text|helpText|description=                    # inline help
learn.?more|Learn.?More                             # learn more links
docs|documentation|help.?center                     # documentation links
info.*icon|InfoIcon|HelpCircle|QuestionMark         # help indicators
```

---

## 4. User Understanding & Navigation

### Information Architecture
- [ ] Navigation is predictable (standard patterns: sidebar, top nav, breadcrumbs)
- [ ] Navigation labels are clear and jargon-free
- [ ] Most important features are in primary navigation (not hidden in settings)
- [ ] Related features are grouped logically
- [ ] Search exists for content-heavy apps

**Grep patterns:**
```
sidebar|Sidebar|nav|Nav|navigation|Navigation       # navigation components
breadcrumb|Breadcrumb                               # breadcrumbs
search|Search|command.*palette|CommandPalette        # search functionality
menu|Menu|dropdown|Dropdown                         # menu components
```

### Task Completion
- [ ] Core task can be completed without leaving the main view
- [ ] Success states celebrate and guide to next action
- [ ] Related actions are suggested after completing a task
- [ ] Undo is available for destructive actions
- [ ] Confirmation dialogs for irreversible actions

**Grep patterns:**
```
success|Success|completed|Completed                 # success states
congrat|Congrat|well.?done|great.?job               # celebrations
next.*step|what.*next|suggested                     # next action guidance
undo|Undo|revert                                    # undo capability
confirm|Confirm|are.?you.?sure                      # confirmation dialogs
```

### Cognitive Load
- [ ] Pages don't present more than 5-7 primary actions
- [ ] Complex features have progressive disclosure
- [ ] Settings are organized into logical sections
- [ ] Defaults are sensible (user doesn't need to configure before using)

---

## 5. Retention Hooks

### Re-engagement Mechanisms
- [ ] Email notifications for important events (not just marketing)
- [ ] In-app notifications exist
- [ ] Digest or summary emails (weekly/monthly)
- [ ] Push notifications (if mobile or PWA)
- [ ] Incomplete action reminders

**Grep patterns:**
```
notification|Notification                           # notification system
digest|summary|weekly.*email                        # digest emails
reminder|Reminder|nudge                             # re-engagement
push.*notification|web.*push|service.?worker        # push notifications
```

### Progress & Achievement
- [ ] Progress indicators for ongoing goals
- [ ] Usage statistics or activity dashboard
- [ ] Streaks, milestones, or achievement badges (if appropriate)
- [ ] Team/collaboration features that create stickiness

**Grep patterns:**
```
progress|Progress|milestone|Milestone               # progress tracking
streak|Streak|badge|Badge|achievement               # gamification
activity|Activity|history|History                    # activity tracking
stats|Stats|analytics|dashboard.*usage              # usage statistics
```

### Habit Formation
- [ ] The app encourages a regular usage pattern (daily, weekly)
- [ ] The app integrates into existing workflows (API, integrations, imports)
- [ ] Data lock-in is appropriate (user's data becomes more valuable over time)
- [ ] Collaboration features increase switching costs

**Grep patterns:**
```
integration|Integration|connect|Connect             # integrations
import|Import|export|Export                          # data portability
api.?key|API.?Key|webhook                           # API access
invite|Invite|team|Team|collaborate                 # collaboration
```

---

## 6. Activation Metrics

### Key Events to Track
- [ ] First meaningful action is defined and trackable
- [ ] Activation event is tracked (e.g., "created first project", "sent first message")
- [ ] Time-to-activation is measurable
- [ ] Activation funnel is defined (signup → setup → first value)

**Grep patterns:**
```
track\(|analytics\.(track|capture|event)            # event tracking
identify\(|analytics\.identify                      # user identification
activation|first.*created|first.*action             # activation events
funnel|conversion|onboarding.*complete              # funnel tracking
```

### What Constitutes Activation
Look for signals of what the app considers a "successful" user:
- Did they create their first resource?
- Did they invite a team member?
- Did they complete a core workflow?
- Did they connect an integration?
- If none of these are tracked, that's a WARNING.

---

## 7. Churn Prevention

### Error Recovery
- [ ] Error pages guide users to recovery (not dead ends)
- [ ] Failed actions offer retry or alternative
- [ ] Missing feature pages suggest workarounds or alternatives
- [ ] Support is easily accessible from error states
- [ ] Session timeout has graceful recovery (don't lose work)

**Grep patterns:**
```
retry|Retry|try.?again                              # retry mechanisms
error.*page|Error.*Page|500|404                     # error pages
session.*expired|timeout|Session.*Timeout           # session handling
auto.?save|autoSave|draft|Draft                     # work preservation
```

### Dead End Prevention
- [ ] No buttons that do nothing (stub handlers)
- [ ] No pages that load blank with no explanation
- [ ] No features that are visible but disabled without explanation
- [ ] No links to non-existent pages

**Grep patterns:**
```
onClick=\{?\(\)\s*=>|onClick=\{?function\(\)\s*\{?\s*\}  # empty click handlers
disabled(?![^>]*title|[^>]*tooltip|[^>]*aria-label)       # disabled without explanation
coming.?soon|Coming.?Soon                                  # placeholder features
```

### Feedback Loops
- [ ] User can report issues from within the app
- [ ] Feature requests can be submitted
- [ ] App asks for feedback at appropriate moments (not immediately)
- [ ] Cancellation flow asks for feedback (if subscription-based)

**Grep patterns:**
```
feedback|Feedback|report.*bug|bug.*report           # feedback mechanisms
feature.*request|suggest|suggestion                  # feature requests
cancel.*reason|churn.*reason|why.*leaving           # cancellation feedback
nps|NPS|satisfaction|rate.*experience               # satisfaction surveys
```

---

## 8. Competitive Positioning

### Messaging Clarity
- [ ] Homepage clearly states who the product is for
- [ ] Product differentiators are specific and verifiable
- [ ] Messaging avoids generic buzzwords without substance
- [ ] Use cases are concrete (not abstract descriptions)

### Positioning Signals in Code
- [ ] Features page or comparison page exists
- [ ] Pricing is competitive and clearly communicated
- [ ] Unique features are highlighted (not buried)
- [ ] Migration or import from competitors exists (if applicable)

**Grep patterns:**
```
migrate|migration|import.*from|switch.*from         # competitor migration
compare|alternative|vs                               # comparison content
use.?case|Use.?Case|for.*teams|for.*developers      # audience targeting
```

---

## Scoring Guide

- **CRITICAL**: Empty dashboard after signup with no guidance, core task takes > 5 minutes to accomplish, UVP is unclear (can't tell what the app does from homepage), dead-end pages/buttons with no functionality
- **WARNING**: No onboarding flow, no empty states with guidance, no activation event tracking, vague UVP without differentiation, no retention mechanisms, missing error recovery paths
- **GOOD**: Clear UVP, first-run experience guides users, core task achievable quickly, empty states are helpful, basic analytics tracking, error states handled
- **EXCELLENT**: Contextual onboarding, activation metrics tracked, retention hooks in place, competitive positioning clear, progressive disclosure, celebration of user milestones, feedback loops active, time-to-value optimized
