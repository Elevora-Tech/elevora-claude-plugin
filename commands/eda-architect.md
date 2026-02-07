---
description: Inngest architecture expert - analyzes your codebase, plans workflow decomposition, recommends flow control patterns, and designs migration strategies from legacy queues to Inngest
allowed-tools: Bash(git:*), Read(/**), Glob(**), Grep(**)
---

Invoke the `eda-architect` skill to perform a full architectural analysis of this repository.

Run in full analysis mode — produce the complete structured report covering:
1. Current architecture summary (Inngest functions and legacy queues)
2. Health check (step hygiene, flow control, error handling, edge cases)
3. Decomposition recommendations
4. Flow control recommendations
5. Migration plan (if legacy queues found)
6. Priority matrix (quick wins vs. strategic changes)
7. Ordered next steps

Pass through any focus area from the user's request (e.g., specific service, directory, or concern).

This is a read-only analysis — do not write or modify any files.
