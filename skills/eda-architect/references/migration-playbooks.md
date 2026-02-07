# Queue-to-Inngest Migration Playbooks

Detailed migration guides from specific queue systems to Inngest. Used by the eda-architect agent when legacy queue code is detected in a codebase.

---

## BullMQ / Bull → Inngest

### Concept Mapping

| BullMQ | Inngest |
|--------|---------|
| `new Queue("email")` | Event type: `"email/send"` |
| `new Worker("email", processor)` | `createFunction({ id: "send-email" }, { event: "email/send" })` |
| `queue.add("send", data)` | `inngest.send({ name: "email/send", data })` |
| `attempts: 3, backoff: { type: "exponential" }` | `retries: 2` (exponential by default) |
| `delay: 60000` | `step.sleep("delay", "1m")` inside function |
| Repeatable jobs (`every: "5m"`) | `{ cron: "*/5 * * * *" }` trigger |
| Job dependencies | `step.invoke()` or `step.waitForEvent()` |
| Job events (`completed`, `failed`) | `onFailure` handler, downstream events |
| Named processors | Separate Inngest functions per job type |
| Rate limiter | `throttle` or `rateLimit` config |
| Concurrency per queue | `concurrency` config with key |

### Migration Steps

1. **Install Inngest SDK** and set up `serve()` handler in your API framework
2. **Map each queue** to an event type name (e.g., `"email"` queue → `"email/send"` event)
3. **Convert each worker processor** to a `createFunction`:
   - Job data → `event.data`
   - `job.progress()` → step boundaries (each step is a progress checkpoint)
   - `job.log()` → `logger` from handler context
4. **Dual-publish phase**: Modify producers to both `queue.add()` and `inngest.send()` simultaneously
5. **Verify parity**: Confirm Inngest functions produce identical results
6. **Cut over**: Remove Bull producer calls, decommission workers
7. **Clean up**: Remove Bull dependencies, Redis queue infrastructure

---

## Celery → Inngest

### Concept Mapping

| Celery | Inngest |
|--------|---------|
| `@app.task` | `createFunction()` |
| `task.delay(args)` | `inngest.send({ name: "task-name", data: { args } })` |
| `task.apply_async(args, countdown=60)` | `step.sleep()` + `step.run()` |
| `chain(task1.s(), task2.s())` | Multi-step function with sequential `step.run()` |
| `chord(group, callback)` | `Promise.all([...steps])` then `step.run("callback")` |
| `group(task.s() for ...)` | `Promise.all([step.run(...) for ...])` |
| Beat schedule | `{ cron: "..." }` trigger |
| `retry(max_retries=3)` | `retries: 3` |
| Result backend | Step return values (memoized automatically) |
| Task routing (queues) | Concurrency keys or separate functions |

### Migration Steps

1. **Set up Inngest** with your Python/Node framework
2. **Map each @task** to an Inngest function:
   - Task arguments → `event.data`
   - Task return value → final `step.run()` return
3. **Convert chains** to sequential steps within one function
4. **Convert chords/groups** to `Promise.all` with parallel steps
5. **Replace beat schedules** with cron triggers
6. **Dual-publish**: Call both `task.delay()` and `inngest.send()` from producers
7. **Verify** results match, then cut over

---

## SQS / SNS → Inngest

### Concept Mapping

| SQS/SNS | Inngest |
|---------|---------|
| `SendMessageCommand` | `inngest.send()` |
| SQS message body | `event.data` |
| Lambda handler (SQS trigger) | `createFunction({ event: "..." })` |
| Dead Letter Queue (DLQ) | `onFailure` handler |
| Visibility timeout | Step-level retries (automatic) |
| Message attributes | `event.data` fields |
| FIFO queue ordering | `concurrency: { limit: 1, key: "event.data.groupId" }` |
| SNS topic → multiple SQS | Multiple functions on same event (fan-out) |
| Message dedup (FIFO) | `idempotency: "event.data.deduplicationId"` |
| Delay queue | `step.sleep()` inside function |

### Migration Steps

1. **Map each SQS queue** to an Inngest event type
2. **Convert Lambda handlers** to Inngest functions:
   - `event.Records[0].body` → `event.data`
   - Error handling → retries config + `onFailure`
3. **Replace DLQ** with `onFailure` handler (can alert, store, or retry manually)
4. **For FIFO queues**: Use `concurrency: { limit: 1, key: "..." }` to maintain ordering
5. **Dual-publish**: Send to both SQS and Inngest during transition
6. **Remove SQS infrastructure** after verification

---

## RabbitMQ → Inngest

### Concept Mapping

| RabbitMQ | Inngest |
|----------|---------|
| Exchange | Event type prefix (e.g., `"orders/"`) |
| Queue | Inngest function |
| Message publish | `inngest.send()` |
| Consumer | `createFunction` handler |
| `channel.ack(msg)` | Step completes successfully |
| `channel.nack(msg)` | Step throws error (triggers retry) |
| Routing key | Event name or `if` filter expression |
| Direct exchange | One event → one function |
| Fanout exchange | One event → multiple functions |
| Topic exchange | Event name patterns + `if` filter expressions |
| Message TTL | `timeouts: { start: "..." }` |
| Prefetch count | `concurrency: { limit: N }` |

### Migration Steps

1. **Map exchanges** to event type namespaces
2. **Map queues** to Inngest functions
3. **Convert routing** to event names + filter expressions:
   - Direct: `{ event: "orders/created" }`
   - Topic: Multiple functions with `if` filters
   - Fanout: Multiple functions on same event
4. **Replace ack/nack** with step success/error
5. **Dual-publish**: `channel.publish()` and `inngest.send()` simultaneously
6. **Decommission** RabbitMQ consumers after verification

---

## Temporal → Inngest

### Concept Mapping

| Temporal | Inngest |
|----------|---------|
| Workflow | Inngest function |
| Activity | `step.run()` |
| `proxyActivities()` | Direct function calls inside `step.run()` |
| Signal | `step.waitForEvent()` |
| Query | Not directly supported (use external state store) |
| Timer (`sleep`) | `step.sleep()` / `step.sleepUntil()` |
| Child workflow | `step.invoke()` |
| Continue-as-new | Send new event + return (no direct equivalent) |
| Retry policy per activity | Per-function retries (not per-step) |
| Search attributes | Event data fields |
| Cron schedule | `{ cron: "..." }` trigger |

### Key Differences

- Temporal has per-activity retry policies; Inngest retries are per-function (all steps share the same retry budget)
- Temporal supports queries on running workflows; Inngest doesn't (use external state)
- Temporal's continue-as-new has no direct equivalent; emit a new event and return
- Temporal activities can be shared across workflows; use `step.invoke()` to call shared Inngest functions

### Migration Steps

1. **Map each workflow** to an Inngest function
2. **Map activities** to `step.run()` calls
3. **Convert signals** to `step.waitForEvent()` with match expressions
4. **Convert child workflows** to `step.invoke()`
5. **Replace timers** with `step.sleep()`/`step.sleepUntil()`
6. **Handle continue-as-new** by emitting new events at workflow boundaries

---

## General Migration Approach: Strangler Fig Pattern

### Phase 1: Dual Publishing
- Modify existing queue producers to also emit Inngest events
- Both old consumers and new Inngest functions process in parallel
- Compare results for correctness

### Phase 2: Shadow Mode
- Inngest functions process events but results are compared, not used
- Log discrepancies for investigation
- Build confidence in the migration

### Phase 3: Gradual Cutover
- Route increasing percentage of traffic to Inngest
- Use feature flags or percentage-based routing
- Monitor error rates and latency

### Phase 4: Decommission
- Remove old queue producer code
- Shut down old consumers/workers
- Remove queue infrastructure (Redis, SQS, RabbitMQ)
- Clean up dependencies

---

## What NOT to Migrate

Not everything belongs in Inngest:

- **Sub-10ms latency paths**: Direct function calls are faster. Inngest adds overhead for durability.
- **Simple pub/sub fanout** where you don't need retries, steps, or coordination — a lightweight event bus may suffice.
- **Stateless transformations** that don't need retry semantics or coordination.
- **High-frequency telemetry/metrics**: Use purpose-built streaming systems (Kafka, Kinesis).
- **Real-time websocket events**: Inngest is for durable execution, not real-time push.
