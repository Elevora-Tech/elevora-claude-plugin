# Inngest Edge Cases & Gotchas

Critical pitfalls catalog for the eda-architect agent. For each issue: what goes wrong, the mistake pattern, the correct approach, and detection signals in a codebase.

---

## 1. Non-Deterministic Code Outside Steps

**What goes wrong**: Code outside `step.run()` re-executes on every function invocation (initial run + every step replay). DB calls, API calls, `Date.now()`, `Math.random()` outside steps produce different results each time.

**Mistake pattern**:
```typescript
async ({ event, step }) => {
  const user = await db.getUser(event.data.userId); // runs EVERY invocation
  const now = Date.now(); // different each time

  await step.run("process", () => doSomething(user, now));
}
```

**Correct approach**:
```typescript
async ({ event, step }) => {
  const user = await step.run("get-user", () => db.getUser(event.data.userId));
  const now = await step.run("get-time", () => Date.now());

  await step.run("process", () => doSomething(user, now));
}
```

**Detection signals**: Any `await` call, `Date.now()`, `new Date()`, `Math.random()`, or `fetch` call at the top level of a handler function (not inside a `step.run`).

---

## 2. Closure Variable Mutation

**What goes wrong**: Assigning to a variable declared outside `step.run()` instead of returning from it. On replay, the step is memoized and its body doesn't execute, so the variable remains `undefined`.

**Mistake pattern**:
```typescript
async ({ event, step }) => {
  let userId;
  await step.run("get-user", async () => {
    userId = await createUser(event.data); // NOT returned
  });
  // userId is undefined on replay
  await step.run("send-email", () => sendEmail(userId));
}
```

**Correct approach**:
```typescript
async ({ event, step }) => {
  const userId = await step.run("get-user", async () => {
    return await createUser(event.data); // returned and memoized
  });
  await step.run("send-email", () => sendEmail(userId));
}
```

**Detection signals**: Variables declared with `let` before a `step.run()` call, then assigned inside the step body without a `return` statement. Look for `let ... = undefined` or `let ...;` followed by `step.run` that assigns but doesn't return.

---

## 3. Multiple Side Effects in One Step

**What goes wrong**: If the second side effect fails, the entire step retries, re-executing the first side effect. This creates duplicates or inconsistent state.

**Mistake pattern**:
```typescript
await step.run("process-payment", async () => {
  await chargeCard(orderId);        // charges again on retry
  await updateOrderStatus(orderId); // if this fails, card re-charged
});
```

**Correct approach**:
```typescript
await step.run("charge-card", () => chargeCard(orderId));
await step.run("update-order", () => updateOrderStatus(orderId));
```

**Detection signals**: Multiple `await` calls to external services or database mutations inside a single `step.run()` callback.

---

## 4. sleepUntil with Dynamic Dates

**What goes wrong**: A date computed inline (not from a prior step) recalculates on each invocation, potentially shifting the wake-up time.

**Mistake pattern**:
```typescript
async ({ event, step }) => {
  const wakeUp = new Date(Date.now() + 86400000); // recalculates each invocation
  await step.sleepUntil("wait", wakeUp);
}
```

**Correct approach**:
```typescript
async ({ event, step }) => {
  const wakeUp = await step.run("calc-wake-time", () => {
    return new Date(Date.now() + 86400000).toISOString();
  });
  await step.sleepUntil("wait", wakeUp);
}
```

**Detection signals**: `step.sleepUntil` called with a value that is not sourced from a prior `step.run()` return. Look for inline `new Date()`, date arithmetic, or variables computed outside steps being passed to `sleepUntil`.

---

## 5. Events Before waitForEvent

**What goes wrong**: If an event is sent before the function reaches its `waitForEvent` call, the event is missed. The function waits indefinitely (until timeout).

**Mistake pattern**:
```typescript
// External service sends "process/complete" immediately
await step.run("start-process", () => startExternalProcess(id));
// But function hasn't reached waitForEvent yet - event is missed
const result = await step.waitForEvent("wait-complete", {
  event: "process/complete",
  match: "data.id",
  timeout: "1h",
});
```

**Correct approach**: Ensure the external process has a delay, or use polling, or design the external system to send the event after a confirmation handshake. Alternatively, use `step.sendEvent` to trigger the external process (ensures ordering within Inngest).

**Detection signals**: `step.run()` that triggers an external process followed immediately by `step.waitForEvent()` for the result of that process. The external process may complete before the wait is registered.

---

## 6. Loop Behavior

**What goes wrong**: Loop bodies re-execute on every function invocation. Only code inside `step.run()` is memoized.

**Mistake pattern**:
```typescript
async ({ event, step }) => {
  const items = event.data.items;
  for (const item of items) {
    console.log(`Processing ${item.id}`); // logs every invocation
    await step.run(`process-${item.id}`, () => process(item));
  }
}
```

**Correct approach**: Minimize code outside steps in loops. Use unique step names per iteration:

```typescript
async ({ event, step }) => {
  const items = event.data.items;
  for (const item of items) {
    await step.run(`process-${item.id}`, () => process(item));
  }
}
```

Or use parallel execution:
```typescript
await Promise.all(items.map(item =>
  step.run(`process-${item.id}`, () => process(item))
));
```

**Detection signals**: `for`/`while`/`forEach` loops containing `step.run()` with significant non-step code inside the loop body.

---

## 7. 4MB Total Step Data Limit

**What goes wrong**: The cumulative size of all step return values exceeds 4MB, causing function failure.

**Mistake pattern**:
```typescript
// Each step returns large data, accumulating beyond 4MB
for (const page of pages) {
  const data = await step.run(`fetch-${page}`, () => fetchPageData(page));
  // data is stored in step memo - accumulates
}
```

**Correct approach**: Return only IDs or references from steps, not full payloads. Store large data externally (S3, database) and pass references:

```typescript
const ref = await step.run("store-data", async () => {
  const data = await fetchLargePayload();
  const key = await s3.upload(data);
  return key; // small reference, not full payload
});
```

**Detection signals**: Steps that return large objects (full API responses, file contents, arrays of records). Functions with many steps each returning data. Look for patterns like `return await fetch(...).then(r => r.json())` in steps processing large datasets.

---

## 8. 1000 Parallel Step Limit

**What goes wrong**: Creating more than 1000 steps (including parallel ones) causes function failure.

**Mistake pattern**:
```typescript
// Processing 5000 items creates 5000 steps
await Promise.all(items.map(item =>
  step.run(`process-${item.id}`, () => process(item))
));
```

**Correct approach**: Batch items within steps, or use fan-out to separate functions:

```typescript
// Batch approach
const chunks = chunkArray(items, 100);
for (const chunk of chunks) {
  await step.run(`process-batch-${chunk[0].id}`, () =>
    Promise.all(chunk.map(item => process(item)))
  );
}

// Or fan-out approach
await step.sendEvent("fan-out", items.map(item => ({
  name: "item/process",
  data: { itemId: item.id },
})));
```

**Detection signals**: `Promise.all` with `step.run` over arrays that could exceed 1000 elements. Dynamic step creation in loops without bounds checking.

---

## 9. Promise.race with optimizeParallelism

**What goes wrong**: With `optimizeParallelism` enabled, `Promise.race` still waits for all steps to complete internally, even though JS resolves the race early. Inngest must execute all steps for determinism.

**Mistake pattern**:
```typescript
// Expects to cancel losing steps, but all execute
const result = await Promise.race([
  step.run("fast-api", () => callFastApi()),
  step.run("slow-api", () => callSlowApi()),
]);
```

**Correct approach**: Use `waitForEvent` with timeout for true race semantics, or accept that all steps will complete and handle at the application level.

**Detection signals**: `Promise.race` used with `step.run` calls, especially in functions with `optimizeParallelism: true`.

---

## 10. Batch Incompatibilities

**What goes wrong**: Combining `batchEvents` with incompatible options silently fails or produces unexpected behavior.

**Incompatible combinations**:
- `batchEvents` + `idempotency` — cannot deduplicate within batches
- `batchEvents` + `rateLimit` — rate limiting applies to batches, not individual events
- `batchEvents` + `cancelOn` — cannot cancel batch runs
- `batchEvents` + `priority` — cannot prioritize batch runs

**Detection signals**: `createFunction` config objects containing `batchEvents` alongside any of the incompatible options.

---

## 11. Retry Count is Per-Step

**What goes wrong**: Developers assume retries: 4 means 5 total attempts for the whole function. Actually, each step gets its own retry budget. A function with 3 steps and 4 retries can attempt up to 15 total step executions.

**Misconception**:
```typescript
// "This function will run at most 5 times" — WRONG
// Each of the 3 steps can retry 4 times independently
inngest.createFunction(
  { id: "my-fn", retries: 4 },
  { event: "test" },
  async ({ step }) => {
    await step.run("step-1", () => mayFail());  // up to 5 attempts
    await step.run("step-2", () => mayFail());  // up to 5 attempts
    await step.run("step-3", () => mayFail());  // up to 5 attempts
  }
);
```

**Detection signals**: Comments or documentation suggesting retries apply to the whole function. Functions with many steps and high retry counts where total attempt count may be surprising.

---

## 12. Step Names Must Be Unique and Stable

**What goes wrong**: Changing a step's name after deployment means the old memoized result is lost. In-flight runs will re-execute the step under the new name.

**Mistake pattern**:
```typescript
// Before: step result memoized as "fetch-data"
await step.run("fetch-data", () => getData());

// After rename: "get-data" is a new step, old "fetch-data" result orphaned
await step.run("get-data", () => getData());
```

**Also problematic**: Dynamic step names that change between invocations:
```typescript
await step.run(`process-${Date.now()}`, () => work()); // different name each time
```

**Detection signals**: Step names containing `Date.now()`, `Math.random()`, or other non-deterministic values. Recent git diffs showing step name changes in existing functions.

---

## 13. inngest.send() vs step.sendEvent()

**What goes wrong**: Using `inngest.send()` inside a function handler instead of `step.sendEvent()`. The send lacks step context, tracing, and memoization. On replay, `inngest.send()` re-sends events.

**Mistake pattern**:
```typescript
async ({ event, step }) => {
  await step.run("process", () => work());
  await inngest.send({ name: "next/step", data: {} }); // re-sends on replay
}
```

**Correct approach**:
```typescript
async ({ event, step }) => {
  await step.run("process", () => work());
  await step.sendEvent("trigger-next", { name: "next/step", data: {} });
}
```

**Detection signals**: `inngest.send()` called inside a `createFunction` handler (not inside `step.run`). Look for the Inngest client instance being used directly within function bodies.

---

## 14. Concurrency Counts Active Steps, Not Functions

**What goes wrong**: Developers think concurrency limits count running function instances. Actually, sleeping, waiting, and invoking functions release their concurrency slot.

**Implication**: A function with `concurrency: { limit: 5 }` can have hundreds of runs in-flight if most are in `sleep`/`waitForEvent` states, with only 5 actively executing steps at any time.

**Detection signals**: Concurrency limits set very high to accommodate sleeping functions (unnecessary). Comments suggesting concurrency controls total in-flight runs.
