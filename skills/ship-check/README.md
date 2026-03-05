# Ship Check - Production Readiness Audit

A Claude Code skill that spawns 7 specialist agents to evaluate whether a web application is ready for real users to pay for and succeed with. Goes beyond code review to answer: **"Can real users pay for and succeed with this app today?"**

## Quick Start

In any repository, run:

```
/ship-check
```

Or focus on a specific domain:

```
/ship-check security
/ship-check performance
/ship-check product
```

## What It Does

1. **Scans your codebase** to detect framework, tech stack, routes, and project structure
2. **Spawns 7 specialist agents** in parallel, each with a detailed reference checklist
3. **Produces a scored report** with a go/no-go verdict, blockers, recommendations, and prioritized action plan

## The 7 Specialists

| Agent | Focus |
|-------|-------|
| Security Auditor | OWASP Top 10, auth, secrets, headers, input validation |
| Feature Completeness Analyst | Routes, CRUD, error/loading/empty states, TODOs |
| UX & Accessibility Inspector | WCAG 2.1 AA, keyboard nav, responsive, mobile, forms |
| Infrastructure & DevOps Reviewer | CI/CD, logging, monitoring, env config, deployment |
| Performance Analyst | Bundle size, images, caching, DB queries, Core Web Vitals |
| Business & Launch Reviewer | SEO, analytics, legal, payments, email, social proof |
| Product Experience Strategist | UVP, onboarding, first-run, differentiation, retention |

## Verdict Scale

| Verdict | Condition |
|---------|-----------|
| Not Production Ready | Any domain is CRITICAL |
| MVP - Needs Work | 2+ domains are WARNING |
| MVP - Ready for Beta | 1 domain WARNING, rest GOOD+ |
| Production Ready | All GOOD or better |
| Launch Ready - Ship It | All EXCELLENT |

## Report Includes

- Domain scorecard with per-domain scores
- Blockers with specific file:line references
- Recommendations grouped by effort level
- Cross-cutting issues spanning multiple domains
- Prioritized action plan (this sprint / before beta / before GA)

## Focus Areas

Run a single domain for faster, targeted analysis:

- `/ship-check security` - Security audit only
- `/ship-check features` - Feature completeness only
- `/ship-check ux` - UX & accessibility only
- `/ship-check infra` - Infrastructure & DevOps only
- `/ship-check performance` - Performance only
- `/ship-check business` - Business & launch readiness only
- `/ship-check product` - Product experience only

## Requirements

- Git repository with source code
- Read-only analysis — this skill never writes or modifies files
