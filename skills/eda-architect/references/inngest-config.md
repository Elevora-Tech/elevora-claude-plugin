# Inngest Configuration Reference

Complete reference for every `createFunction` configuration option. Used by the eda-architect agent when recommending flow control and function configuration.

---

## Function Identity

### id (required)
- **Type**: `string`
- **What**: Unique identifier for the function. Used in logging, dashboard, and invocation.
- **Format**: Lowercase, alphanumeric, hyphens. e.g., `"process-order"`, `"send-welcome-email"`
- **Gotcha**: Changing `id` creates a new function; the old one becomes orphaned.

### name (optional)
- **Type**: `string`
- **What**: Human-readable display name shown in the Inngest dashboard.
- **Default**: Derived from `id`

---

## Retry Configuration

### retries
- **Type**: `number` (0-20)
- **Default**: `4` (meaning up to 5 total attempts: 1 initial + 4 retries)
- **What**: Number of retry attempts after initial failure
- **Behavior**: Retries are per-step, not per-function. A function with 3 steps and 4 retries can have up to 15 total step attempts.
- **Backoff**: Exponential by default
- **When to use**: Most functions should keep the default. Set to `0` for functions that must not retry (e.g., sending notifications where duplicates are harmful).
- **When NOT to use**: Don't set high retries for functions calling rate-limited APIs — use `RetryAfterError` instead.
- **Common values**: `0` (no retries), `4` (default), `10` (for flaky external services), `20` (maximum, for critical paths)

---

## Concurrency

### concurrency
- **Type**: `object | object[]` (up to 2 limits)
- **What**: Limits how many function runs execute simultaneously

```typescript
{
  concurrency: {
    limit: 10,
    key: "event.data.tenantId",   // optional: per-key limit
    scope: "fn",                   // optional: "fn" | "env" | "account"
  }
}
```

- **limit**: Max concurrent runs (required)
- **key**: CEL expression to create separate limits per key value. Common: `"event.data.tenantId"`, `"event.data.userId"`
- **scope**: Where the limit applies
  - `"fn"` (default): This function only
  - `"env"`: Across all functions in the environment
  - `"account"`: Across all environments

**When to use**: Multi-tenant fairness (per-tenant key), database connection protection (global limit), external API concurrent request limits.

**When NOT to use**: If you need to drop excess work (use rateLimit). If you need time-based limits (use throttle).

**Multiple limits**: Up to 2 concurrency limits for layered control:
```typescript
{
  concurrency: [
    { limit: 100 },                              // 100 total
    { limit: 5, key: "event.data.tenantId" },    // max 5 per tenant
  ]
}
```

**Gotcha**: Sleep, waitForEvent, and invoke do NOT consume concurrency slots. Only actively executing steps count.

---

## Throttle

### throttle
- **Type**: `object`
- **What**: Soft rate limit — excess runs are enqueued for later processing

```typescript
{
  throttle: {
    limit: 30,
    period: "1m",
    burst: 5,               // optional: allow initial burst
    key: "event.data.api",  // optional: per-key throttle
  }
}
```

- **limit**: Max runs started per period
- **period**: Time window (`"1s"`, `"1m"`, `"1h"`, `"1d"`)
- **burst**: Allow this many extra runs immediately before throttling kicks in
- **key**: CEL expression for per-key throttling

**When to use**: External API rate limits where you want eventual processing. All excess work should still be done, just slower.

**When NOT to use**: If excess work should be dropped entirely (use rateLimit). If you need concurrent execution limits (use concurrency).

**Gotcha**: Throttled runs stay in the queue and will process eventually. Queue can grow large under sustained load.

---

## Rate Limit

### rateLimit
- **Type**: `object`
- **What**: Hard cap — excess runs are dropped entirely

```typescript
{
  rateLimit: {
    limit: 1,
    period: "5s",
    key: "event.data.userId",  // optional
  }
}
```

- **limit**: Max runs per period
- **period**: Time window
- **key**: CEL expression for per-key rate limiting

**When to use**: When excess work is truly unwanted. Webhook dedup (process only once per period), preventing abuse.

**When NOT to use**: If excess work should be processed eventually (use throttle). If you want collapse semantics (use debounce).

**Incompatibility**: Cannot combine with `batchEvents`.

---

## Debounce

### debounce
- **Type**: `object`
- **What**: Collapses rapid-fire events into a single run. Only the last event in the window triggers execution.

```typescript
{
  debounce: {
    period: "5s",
    key: "event.data.userId",  // optional
  }
}
```

- **period**: Window to wait for more events before executing
- **key**: CEL expression for per-key debouncing

**When to use**: Search-as-you-type, save-on-idle, dedup rapid triggers where only the latest state matters.

**When NOT to use**: If all events should be processed (use throttle). If you need to drop excess entirely (use rateLimit).

---

## Idempotency

### idempotency
- **Type**: `string` (CEL expression)
- **What**: Prevents duplicate processing. Events matching the same idempotency key within 24 hours only execute once.

```typescript
{
  idempotency: "event.data.paymentId"
}
```

**When to use**: Payment processing, webhook dedup where the same event might arrive multiple times, any operation that must execute exactly once per unique key.

**When NOT to use**: When events with the same key legitimately need separate processing.

**Gotcha**: 24-hour window only. Cannot extend or shorten.

**Incompatibility**: Cannot combine with `batchEvents`.

---

## Priority

### priority
- **Type**: `object`
- **What**: Dynamically order queued runs. Higher priority runs execute first.

```typescript
{
  priority: {
    run: "event.data.plan == 'enterprise' ? 200 : event.data.plan == 'pro' ? 100 : 0"
  }
}
```

- **run**: CEL expression that returns a number from -600 to 600

**When to use**: Paid-tier prioritization, SLA-based ordering, VIP customer priority.

**When NOT to use**: If runs should be processed FIFO (the default).

**Incompatibility**: Cannot combine with `batchEvents`.

---

## Batch Events

### batchEvents
- **Type**: `object`
- **What**: Collects multiple events and processes them in a single function run

```typescript
{
  batchEvents: {
    maxSize: 100,
    timeout: "10s",
    key: "event.data.table",  // optional: separate batches by key
  }
}
```

- **maxSize**: Max events per batch (required)
- **timeout**: Max wait time for batch to fill (required)
- **key**: CEL expression to group events into separate batches

**When to use**: Bulk database writes, reducing API calls, aggregating metrics.

**When NOT to use**: When each event needs individual handling or individual error recovery.

**Incompatibilities**: Cannot combine with `idempotency`, `rateLimit`, `cancelOn`, or `priority`.

**Gotcha**: Handler receives `events` (array) instead of `event`. Max 10MB total payload.

---

## Cancel On

### cancelOn
- **Type**: `object[]`
- **What**: Cancel a running function when a specified event arrives

```typescript
{
  cancelOn: [{
    event: "cart/updated",
    match: "data.cartId",     // optional: correlate events
    if: "async.data.action == 'abandoned'",  // optional: CEL condition
    timeout: "1h",            // optional: stop listening after timeout
  }]
}
```

**When to use**: Cancel stale runs when superseded (cart updates, duplicate requests), cancel pending workflows on user action.

**When NOT to use**: When you need to handle cancellation with cleanup logic (use waitForEvent + manual handling instead).

**Incompatibility**: Cannot combine with `batchEvents`.

---

## On Failure

### onFailure
- **Type**: `function`
- **What**: Handler called when all retries are exhausted

```typescript
inngest.createFunction(
  {
    id: "critical-process",
    retries: 5,
    onFailure: async ({ event, error, step }) => {
      await step.run("alert-team", () => sendAlert(error.message));
      await step.run("log-failure", () => logToDeadLetter(event));
    },
  },
  { event: "critical/process" },
  async ({ event, step }) => { /* main handler */ }
);
```

**When to use**: Cleanup after permanent failure, alerting, dead letter queue logic, compensating transactions.

---

## Timeouts

### timeouts
- **Type**: `object`
- **What**: Time limits for function lifecycle

```typescript
{
  timeouts: {
    start: "1h",    // max time from trigger to first execution
    finish: "24h",  // max total execution time including sleeps
  }
}
```

- **start**: Cancel if function hasn't started within this time
- **finish**: Cancel if function hasn't completed within this time (including sleep/wait time)

---

## Trigger Options

### Event Trigger

```typescript
{ event: "user/signup" }

// With filter
{ event: "user/signup", if: "event.data.plan == 'enterprise'" }
```

- `if`: CEL expression to filter which events trigger this function

### Cron Trigger

```typescript
{ cron: "0 9 * * MON-FRI" }

// With timezone
{ cron: "TZ=America/New_York 0 9 * * MON-FRI" }
```

### Multiple Triggers

A function can have multiple triggers:
```typescript
[
  { event: "user/signup" },
  { event: "user/imported" },
  { cron: "0 0 * * *" },
]
```

---

## Handler Context

The handler function receives a context object:

| Property | Description |
|----------|-------------|
| `event` | The triggering event (single event or first in batch) |
| `events` | Array of events (only with batchEvents) |
| `step` | Step toolbox (run, sleep, sleepUntil, waitForEvent, sendEvent, invoke) |
| `runId` | Unique identifier for this function run |
| `logger` | Structured logger |
| `attempt` | Current attempt number (0-indexed) |
