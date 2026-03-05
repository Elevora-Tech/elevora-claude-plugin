---
description: Production readiness audit - spawns 7 specialist agents to evaluate security, features, UX, infra, performance, business readiness, and product experience for any web app
allowed-tools: Bash(git:*), Read(/**), Glob(**), Grep(**), Agent
argument-hint: '[focus-area]'
---

Invoke the `ship-check` skill to perform a comprehensive production readiness audit of this repository.

Run in full audit mode — spawn all 7 specialist agents and produce the complete structured report covering:
1. Security audit (OWASP, auth, secrets, headers)
2. Feature completeness (routes, CRUD, error/loading/empty states)
3. UX & accessibility (WCAG, keyboard nav, responsive, mobile)
4. Infrastructure & DevOps (CI/CD, logging, monitoring, deployment)
5. Performance (bundle size, images, caching, Core Web Vitals)
6. Business & launch readiness (SEO, analytics, legal, payments)
7. Product experience (onboarding, UVP, first-run, differentiation)

If the user specified a focus area, narrow to that domain but still produce the full report structure.

This is a read-only analysis — do not write or modify any files.
