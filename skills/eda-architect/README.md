# EDA Architect - Inngest Expert Architect & Planner

A Claude Code skill that acts as a senior Inngest architect. Analyzes your codebase, plans workflow decomposition, recommends flow control patterns, and designs migration strategies from legacy queues to Inngest.

## Quick Start

In any repository, run:

```
/eda-architect
```

Or with natural language:
- "Should this be one Inngest function or multiple?"
- "How should I structure this workflow in Inngest?"
- "Migrate this BullMQ code to Inngest"
- "Review my Inngest functions for issues"
- "What flow control should I use here?"
- "Is this sleepUntil safe?"
- "Plan the Inngest architecture for this feature"

## What It Does

1. **Scans your codebase** for Inngest functions and legacy queue systems (BullMQ, Celery, SQS, RabbitMQ, Temporal, Sidekiq)
2. **Analyzes existing Inngest functions** for step hygiene, flow control fitness, error handling, and edge case violations
3. **Recommends decomposition** — single function vs. multiple coordinated functions with reasoning
4. **Suggests flow control** — which config options to use (concurrency, throttle, rateLimit, debounce, idempotency, priority, batch)
5. **Plans migrations** from legacy queues to Inngest with concept mappings and step-by-step guides
6. **Identifies edge cases** — 14 known Inngest gotchas checked against your code

## Modes

- **Slash command** (`/eda-architect`): Full structured report with health check, recommendations, and priority matrix
- **Natural language**: Conversational analysis with targeted answers and follow-up questions

## Output Includes

- Current architecture summary (Inngest functions, legacy queues, communication patterns)
- Health check (step hygiene issues, flow control misuse, missing error handling)
- Decomposition recommendations with before/after architecture
- Flow control recommendations per function
- Migration plan (if legacy queues found)
- Priority matrix (quick wins vs. strategic changes)
- Ordered next steps with file/function references

## Requirements

- Git repository with source code
- Read-only analysis — this skill never writes or modifies files

## Supported Queue Systems

| System | Detection |
|--------|-----------|
| Inngest (existing) | `createFunction`, `new Inngest(`, `serve()` |
| BullMQ / Bull | `Queue`, `Worker`, `.process()` |
| Celery | `@app.task`, `.delay()`, `apply_async` |
| SQS / SNS | `SQSClient`, `SendMessageCommand` |
| RabbitMQ | `amqplib`, `channel.publish`, `channel.consume` |
| Temporal | `@temporalio/client`, `proxyActivities` |
| Sidekiq / Resque | `perform_async`, `sidekiq` |

## Use Cases

- **New to Inngest**: "How should I structure Inngest functions for this feature?"
- **Code review**: "Review my Inngest functions for issues and edge cases"
- **Migration planning**: "We use BullMQ — plan the migration to Inngest"
- **Architecture decisions**: "Should this be one function or multiple?"
- **Flow control selection**: "What flow control config should I use?"
- **Edge case detection**: "Is my sleepUntil / waitForEvent / loop pattern safe?"
