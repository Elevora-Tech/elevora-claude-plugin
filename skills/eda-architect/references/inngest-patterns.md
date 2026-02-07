# Inngest Patterns & Best Practices

Reference for common Inngest patterns the eda-architect agent draws from when analyzing codebases.

---

## Multi-Step Workflows

### Sequential Steps with Data Passing

Each `step.run()` executes exactly once (memoized on replay). Return values from steps are the only safe way to pass data between them.

```typescript
const result = await inngest.createFunction(
  { id: "process-order" },
  { event: "order/placed" },
  async ({ event, step }) => {
    const validated = await step.run("validate-order", async () => {
      return validateOrder(event.data.orderId);
    });

    const charged = await step.run("charge-payment", async () => {
      return chargeCard(validated.paymentMethod, validated.total);
    });

    await step.run("send-confirmation", async () => {
      await sendEmail(validated.email, charged.receiptId);
    });
  }
);
```

### Single Side Effect Per Step

Each step should contain exactly one side effect. If a step contains two side effects and the second fails, the first re-executes on retry because the step is atomic.

**Wrong**: Two API calls in one step
```typescript
await step.run("notify", async () => {
  await sendEmail(user.email);    // runs again on retry
  await sendSlack(user.slackId);  // if this fails, email re-sends
});
```

**Right**: Separate steps
```typescript
await step.run("send-email", async () => {
  await sendEmail(user.email);
});
await step.run("send-slack", async () => {
  await sendSlack(user.slackId);
});
```

---

## Parallel Execution

### Promise.all Pattern

Run independent steps concurrently within a single function:

```typescript
const [user, inventory, pricing] = await Promise.all([
  step.run("fetch-user", () => getUser(event.data.userId)),
  step.run("check-inventory", () => checkStock(event.data.sku)),
  step.run("get-pricing", () => calculatePrice(event.data.sku)),
]);
```

### optimizeParallelism Flag

Enables steps to execute before prior steps complete (SDK v3.27+):

```typescript
inngest.createFunction(
  { id: "pipeline", optimizeParallelism: true },
  { event: "data/process" },
  async ({ step }) => {
    // These all start immediately, not sequentially
    const a = await step.run("step-a", () => fetchA());
    const b = await step.run("step-b", () => fetchB());
    const c = await step.run("step-c", () => fetchC());
  }
);
```

### Limits

- **1000 steps maximum** per function invocation (including parallel)
- **4MB total step data** across all step return values combined
- Exceeding either limit causes function failure

---

## Fan-Out

### Multiple Functions on Same Event

Different functions independently trigger on the same event. Each has its own reliability, retries, and concurrency:

```typescript
// Function A
inngest.createFunction(
  { id: "send-welcome-email" },
  { event: "user/signup" },
  async ({ event, step }) => { /* ... */ }
);

// Function B - independent
inngest.createFunction(
  { id: "provision-account" },
  { event: "user/signup" },
  async ({ event, step }) => { /* ... */ }
);
```

### Dynamic Fan-Out with step.sendEvent

Emit events from within a function to trigger downstream functions:

```typescript
await step.sendEvent("fan-out-processing", {
  name: "item/process",
  data: items.map(item => ({ itemId: item.id })),
});
```

Use `step.sendEvent()` (not `inngest.send()`) inside functions for proper context and tracing.

---

## Function Invocation

### step.invoke for RPC-like Calls

Call another Inngest function and await its result. Useful when the called function needs different retry/concurrency config:

```typescript
const result = await step.invoke("process-payment", {
  function: processPaymentFn,
  data: { orderId: order.id, amount: order.total },
});
```

### referenceFunction for Cross-App Invocation

Invoke functions from other Inngest apps:

```typescript
const externalFn = referenceFunction<typeof externalApp>({
  functionId: "external-app-process",
  appId: "external-app",
});

const result = await step.invoke("call-external", {
  function: externalFn,
  data: { id: "123" },
});
```

### When to Use invoke vs fan-out

| Aspect | step.invoke | step.sendEvent |
|--------|-------------|----------------|
| Result access | Yes - awaits result | No - fire and forget |
| Retry isolation | Yes - called fn has own retries | Yes |
| Concurrency isolation | Yes - separate config | Yes |
| Cross-app | Yes (with referenceFunction) | Yes (via event routing) |
| Scale | One-to-one | One-to-many |

---

## Event Coordination

### step.waitForEvent

Pause execution until a matching event arrives or timeout:

```typescript
const approval = await step.waitForEvent("wait-for-approval", {
  event: "order/approved",
  match: "data.orderId",  // correlate by orderId
  timeout: "24h",
});

if (!approval) {
  // Timeout - no approval received
  await step.run("escalate", () => notifyManager(orderId));
} else {
  await step.run("fulfill", () => fulfillOrder(approval.data.orderId));
}
```

### match vs if Expressions

- `match`: shorthand for matching a field between trigger event and waited event (`"data.orderId"` means both events must have same `data.orderId`)
- `if`: CEL expression for complex conditions (`"event.data.orderId == async.data.orderId && async.data.status == 'approved'"`)

### Race Condition Awareness

Events sent **before** the function reaches `waitForEvent` are missed. Design systems so the waited event is triggered after the function is running:

```typescript
// WRONG: event might fire before waitForEvent
await inngest.send({ name: "start/external-process", data: { id } });
const result = await step.waitForEvent("wait-result", {
  event: "external/complete",
  match: "data.id",
  timeout: "1h",
});

// RIGHT: use step.sendEvent to ensure ordering
await step.sendEvent("trigger-external", {
  name: "start/external-process",
  data: { id },
});
const result = await step.waitForEvent("wait-result", {
  event: "external/complete",
  match: "data.id",
  timeout: "1h",
});
```

---

## Scheduled Work

### Cron Triggers

```typescript
inngest.createFunction(
  { id: "daily-cleanup" },
  { cron: "TZ=America/New_York 0 9 * * MON-FRI" },
  async ({ step }) => {
    await step.run("cleanup", () => cleanupStaleRecords());
  }
);
```

### step.sleep / step.sleepUntil

```typescript
await step.sleep("wait-before-reminder", "3d");

// Or sleep until a specific time (must come from a prior step)
const scheduledTime = await step.run("get-time", () => getScheduledTime());
await step.sleepUntil("wait-until-scheduled", scheduledTime);
```

**Important**: Dynamic dates for `sleepUntil` must come from a prior `step.run()` return value, not computed inline (which re-evaluates on each invocation).

---

## Human-in-the-Loop

### Approval Workflows

```typescript
inngest.createFunction(
  { id: "expense-approval" },
  { event: "expense/submitted" },
  async ({ event, step }) => {
    await step.run("notify-manager", async () => {
      await sendApprovalRequest(event.data.managerId, event.data.expenseId);
    });

    const decision = await step.waitForEvent("wait-approval", {
      event: "expense/decided",
      match: "data.expenseId",
      timeout: "7d",
    });

    if (!decision) {
      await step.run("auto-escalate", () => escalate(event.data.expenseId));
    } else if (decision.data.approved) {
      await step.run("process-payment", () => reimburse(event.data.expenseId));
    } else {
      await step.run("notify-rejection", () => notifyRejected(event.data.expenseId));
    }
  }
);
```

---

## Drip Campaigns

### Sleep + waitForEvent Combos

```typescript
inngest.createFunction(
  { id: "user-onboarding" },
  { event: "user/signup" },
  async ({ event, step }) => {
    await step.run("send-welcome", () => sendWelcomeEmail(event.data.email));

    await step.sleep("wait-day-1", "1d");

    const completed = await step.waitForEvent("check-profile", {
      event: "user/profile-completed",
      match: "data.userId",
      timeout: "0s", // check immediately, don't wait
    });

    if (!completed) {
      await step.run("send-reminder", () => sendProfileReminder(event.data.email));
    }

    await step.sleep("wait-day-3", "3d");

    await step.run("send-tips", () => sendTipsEmail(event.data.email));
  }
);
```

---

## Cancellation

### cancelOn for Superseding Events

Cancel a running function when a newer event makes it obsolete:

```typescript
inngest.createFunction(
  {
    id: "process-cart-checkout",
    cancelOn: [{ event: "cart/updated", match: "data.cartId" }],
  },
  { event: "cart/checkout-started" },
  async ({ event, step }) => {
    await step.sleep("wait-for-final-changes", "5m");
    await step.run("process", () => processCheckout(event.data.cartId));
  }
);
```

When `cart/updated` fires with the same `cartId`, the running function is cancelled and a new one can start.

---

## Batch Processing

### batchEvents for Bulk Operations

Collect multiple events and process them together:

```typescript
inngest.createFunction(
  {
    id: "bulk-db-insert",
    batchEvents: {
      maxSize: 100,
      timeout: "10s",
      key: "event.data.table",
    },
  },
  { event: "record/insert" },
  async ({ events, step }) => {
    // events is an array of up to 100 events
    await step.run("bulk-insert", async () => {
      await db.batchInsert(events.map(e => e.data.record));
    });
  }
);
```

### Configuration

- `maxSize`: Max events per batch (required)
- `timeout`: Max wait time before processing partial batch (required)
- `key`: CEL expression to group events into separate batches (optional)

### Limits & Incompatibilities

- **10MB total** batch payload limit
- Cannot combine batchEvents with: `idempotency`, `rateLimit`, `cancelOn`, `priority`
- Handler receives `events` (array) instead of `event` (singular)
