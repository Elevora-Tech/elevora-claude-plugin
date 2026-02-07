# Inngest Decision Frameworks

Expanded decision trees with worked examples for the eda-architect agent. These frameworks drive the core analytical value of architectural recommendations.

---

## Framework 1: Function Decomposition

### Decision Tree: Single Function vs. Multiple Coordinated Functions

```
Is shared state needed between stages?
├── Yes → Do stages need different retry/concurrency policies?
│   ├── Yes → Multiple functions with step.invoke()
│   └── No → Will total steps exceed 1000?
│       ├── Yes → Multiple functions (orchestrator + workers)
│       └── No → Will total step data exceed 4MB?
│           ├── Yes → Multiple functions (externalize data)
│           └── No → Single function with multiple steps
├── No → Are stages independently reusable?
│   ├── Yes → Separate functions triggered by events
│   └── No → Do different teams own different stages?
│       ├── Yes → Separate functions (team boundaries)
│       └── No → Is fan-out to variable recipients needed?
│           ├── Yes → Orchestrator + fan-out via sendEvent
│           └── No → Single function
```

### Use Single Function When:
- Steps are tightly coupled and share data between them
- Linear workflow with sequential dependencies
- Total steps < 1000 and total data < 4MB
- Same retry/concurrency policy for all stages
- One team owns the entire workflow

### Use Multiple Functions When:
- Logic is independently reusable across workflows
- Different retry or concurrency needs per stage
- Different teams own different stages
- More than 1000 steps needed (split into sub-functions)
- Fan-out to dynamic number of recipients
- Stages have very different execution profiles (fast vs. slow)

### Hybrid: Orchestrator Pattern
Use an orchestrator function that calls sub-functions via `step.invoke()`:
- Orchestrator manages workflow logic, sequencing, and error handling
- Sub-functions handle specific tasks with their own config
- Best of both: shared orchestration + isolated execution

### Worked Example: E-Commerce Order Flow

**Workflow**: Order placed → validate → charge payment → update inventory → send confirmation → schedule delivery

**Analysis**:
- Validate + charge: tightly coupled, same retry policy → single function
- Update inventory: different concurrency needs (protect DB) → separate function
- Send confirmation: independent, different retry policy (email) → separate function
- Schedule delivery: independent, cron-like checks needed → separate function

**Recommendation**:
```
orchestrate-order (orchestrator)
  ├── step.run("validate")
  ├── step.run("charge-payment")
  ├── step.invoke(update-inventory-fn)
  ├── step.invoke(send-confirmation-fn)
  └── step.invoke(schedule-delivery-fn)
```

### Worked Example: User Onboarding Drip Campaign

**Workflow**: Signup → welcome email → wait 1d → check profile → reminder if incomplete → wait 3d → tips email → wait 7d → survey

**Analysis**:
- All steps are sequential with sleeps between them
- Shared state (userId) throughout
- Should cancel if user unsubscribes
- Total steps < 10, data < 4MB

**Recommendation**: Single function with cancelOn
```
user-onboarding (single function)
  cancelOn: [{ event: "user/unsubscribed", match: "data.userId" }]
  ├── step.run("send-welcome")
  ├── step.sleep("1d")
  ├── step.run("check-profile")
  ├── conditional: step.run("send-reminder")
  ├── step.sleep("3d")
  ├── step.run("send-tips")
  ├── step.sleep("7d")
  └── step.run("send-survey")
```

---

## Framework 2: Step Coordination Selection

### Comparison Matrix

| Aspect | step.run | step.invoke | step.sendEvent |
|--------|----------|-------------|----------------|
| **Result access** | Yes (inline) | Yes (awaited) | No (fire-and-forget) |
| **Retry isolation** | No (shares fn retries) | Yes (own retries) | Yes (own retries) |
| **Concurrency isolation** | No (shares fn concurrency) | Yes (own concurrency) | Yes (own concurrency) |
| **Cross-app calls** | No | Yes (referenceFunction) | Yes (event routing) |
| **Scale limit** | 1000 steps per fn | No limit (separate fns) | No limit (separate fns) |
| **Tracing** | Same trace | Linked trace | Separate traces |
| **Error handling** | Fails parent fn | Can catch StepError | Independent |

### When to Use Each

**step.run**: Default choice for inline work
- Simple operations within a workflow
- When you need the result immediately
- When shared retry/concurrency config is fine
- Examples: API call, DB query, data transformation, sending a notification

**step.invoke**: RPC-like call to another function
- When the called function needs different retry policy
- When the called function needs different concurrency limits
- When the function is reused across multiple callers
- When calling functions in other Inngest apps
- Examples: payment processing (needs strict concurrency), shared validation logic

**step.sendEvent (fan-out)**: Fire-and-forget to one or many
- When you don't need the result in the caller
- When multiple independent functions should react
- When you need unlimited scale (no 1000-step limit)
- When the downstream work is truly independent
- Examples: notify multiple services of a state change, trigger batch processing

---

## Framework 3: Flow Control Selection

### Decision Flowchart

```
Do excess runs need to be handled?
├── No → No flow control needed (default behavior)
└── Yes → Should excess runs eventually execute?
    ├── Yes → Are events duplicated rapidly?
    │   ├── Yes → Only latest event matters?
    │   │   ├── Yes → debounce
    │   │   └── No → Should dedup by key?
    │   │       ├── Yes → idempotency
    │   │       └── No → throttle (soft rate limit)
    │   └── No → throttle
    └── No (excess should be dropped) → rateLimit

Need to limit parallel execution?
├── Per-tenant fairness? → concurrency with key
├── Global resource protection? → concurrency (global limit)
└── Time-based rate? → throttle or rateLimit

Need to prioritize certain runs?
└── Yes → priority (with run expression)

Need to batch events for efficiency?
└── Yes → batchEvents (check incompatibilities)
```

### Incompatibility Matrix

| Option | concurrency | throttle | rateLimit | debounce | idempotency | priority | batchEvents | cancelOn |
|--------|-------------|----------|-----------|----------|-------------|----------|-------------|----------|
| **concurrency** | - | OK | OK | OK | OK | OK | OK | OK |
| **throttle** | OK | - | OK | OK | OK | OK | OK | OK |
| **rateLimit** | OK | OK | - | OK | OK | OK | **NO** | OK |
| **debounce** | OK | OK | OK | - | OK | OK | OK | OK |
| **idempotency** | OK | OK | OK | OK | - | OK | **NO** | OK |
| **priority** | OK | OK | OK | OK | OK | - | **NO** | OK |
| **batchEvents** | OK | OK | **NO** | OK | **NO** | **NO** | - | **NO** |
| **cancelOn** | OK | OK | OK | OK | OK | OK | **NO** | - |

### Quick Reference: Which Flow Control?

- **"Process max N per minute"** → `throttle: { limit: N, period: "1m" }`
- **"Only N at the same time"** → `concurrency: { limit: N }`
- **"Max N per tenant at once"** → `concurrency: { limit: N, key: "event.data.tenantId" }`
- **"Drop excess beyond N/minute"** → `rateLimit: { limit: N, period: "1m" }`
- **"Only process latest event"** → `debounce: { period: "5s" }`
- **"Never process same ID twice"** → `idempotency: "event.data.id"`
- **"Paid users first"** → `priority: { run: "event.data.plan == 'paid' ? 100 : 0" }`
- **"Batch writes for efficiency"** → `batchEvents: { maxSize: 100, timeout: "10s" }`

---

## Framework 4: Error Strategy

### Decision Tree

```
What type of error occurred?
├── Transient (network timeout, 503, connection reset)
│   └── Use default retries (exponential backoff)
│       └── Still failing after all retries?
│           └── onFailure handler for alerting/dead letter
├── Permanent (400 bad request, validation failure, business rule violation)
│   └── Throw NonRetriableError (skip remaining retries)
│       └── onFailure handler for cleanup if needed
├── Rate limited (429 too many requests)
│   └── Throw RetryAfterError with delay from response headers
│       └── Inngest retries after specified delay
├── Partial success (some items in batch succeeded, some failed)
│   └── Return success for processed items
│       └── Emit new event for failed items (separate retry path)
└── Upstream dependency down
    └── step.waitForEvent for health check event + step.sleep for polling
        └── Timeout → onFailure with alerting
```

### Error Types Reference

| Error Type | Class | Behavior |
|-----------|-------|----------|
| Default (any throw) | `Error` | Retries up to configured limit with exponential backoff |
| Non-retriable | `NonRetriableError` | Immediately fails, skips remaining retries, triggers onFailure |
| Retry after delay | `RetryAfterError` | Retries after specified duration (for rate limits) |
| Step error (catch) | `StepError` | Catch errors from `step.invoke()` for fallback logic |

### Error Handling Code Patterns

**NonRetriableError for permanent failures**:
```typescript
import { NonRetriableError } from "inngest";

await step.run("validate", () => {
  if (!isValid(data)) {
    throw new NonRetriableError("Invalid data: missing required fields");
  }
  return data;
});
```

**RetryAfterError for rate limits**:
```typescript
import { RetryAfterError } from "inngest";

await step.run("call-api", async () => {
  const res = await fetch(apiUrl);
  if (res.status === 429) {
    const retryAfter = res.headers.get("Retry-After") || "60";
    throw new RetryAfterError(`Rate limited`, parseInt(retryAfter) * 1000);
  }
  return res.json();
});
```

**StepError catch for fallback logic**:
```typescript
try {
  const result = await step.invoke("call-primary", {
    function: primaryServiceFn,
    data: payload,
  });
  return result;
} catch (err) {
  if (err instanceof StepError) {
    // Primary service failed, try fallback
    return await step.invoke("call-fallback", {
      function: fallbackServiceFn,
      data: payload,
    });
  }
  throw err;
}
```

**onFailure for cleanup and alerting**:
```typescript
inngest.createFunction(
  {
    id: "process-payment",
    retries: 5,
    onFailure: async ({ event, error, step }) => {
      await step.run("alert", () =>
        sendSlackAlert(`Payment failed: ${error.message}`)
      );
      await step.run("compensate", () =>
        releaseHold(event.data.holdId)
      );
    },
  },
  { event: "payment/process" },
  async ({ event, step }) => { /* main logic */ }
);
```

---

## Applying Frameworks: Analysis Checklist

When analyzing a codebase, apply frameworks in this order:

1. **Decomposition (Framework 1)**: For each workflow/function, assess whether it should be one function or many
2. **Coordination (Framework 2)**: For each multi-function interaction, determine the right coordination primitive
3. **Flow Control (Framework 3)**: For each function, assess whether flow control is needed and which type
4. **Error Strategy (Framework 4)**: For each function, verify the error handling is appropriate for the failure modes

Flag mismatches between the current implementation and framework recommendations as actionable findings.
