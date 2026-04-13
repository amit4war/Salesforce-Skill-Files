---
name: async-apex-patterns
description: "Design and implement Apex classes and Asynchronous Apex patterns: future methods, Queueable, Batch Apex, and Schedulable. Use when encapsulating business logic in service/domain classes, offloading work that exceeds synchronous governor limits, performing external callouts from triggers, processing large record volumes with Batch Apex, chaining jobs with Queueable, scheduling recurring background jobs, or handling mixed-DML errors by separating transactions. Covers pattern selection, bulkification, recursion prevention, chaining depth limits, concurrency limits, and test strategies for all async types."
argument-hint: "<pattern type: future|queueable|batch|schedulable> <business logic summary> <data volume estimate>"
---

# SKILL: Apex Class and Asynchronous Apex

## Purpose

Design and implement Apex classes and Asynchronous Apex patterns to encapsulate business logic and handle operations that need to run outside the standard request-response cycle. Asynchronous Apex (future methods, Queueable, Batch, Schedulable) allows processing tasks in the background when system resources are available, enabling scalable solutions that stay within Salesforce governor limits.

## When to Use

- **Encapsulating business logic:** Use Apex classes to organize and reuse complex logic (e.g., an `InvoiceService` class) — the foundation of domain and service layer patterns.
- **Avoiding synchronous limits:** Use async Apex when a process might exceed the 10-second CPU limit or other governor limits, especially for large data volumes.
- **External callouts:** Use future or queueable when performing callouts to external APIs (callouts are not allowed after DML in the same transaction).
- **Delayed or scheduled execution:** Use Schedulable when business requirements dictate code runs at a specific time (nightly maintenance, weekly summaries).
- **Better user experience:** Offload heavy processing to async so the initial transaction stays fast and the UI remains responsive.
- **Chained/dependent processes:** Use Queueable when steps must run sequentially — one job starts the next on completion.

## Required Inputs

- **Functional requirements:** What business logic should the class implement or what process should the async job perform?
- **Triggering context:** How will it be invoked (trigger, scheduled time, API call, etc.)? This determines whether sync or async is appropriate.
- **Data scope and volume:** Estimate record counts — informs whether Batch Apex (large volumes) or a simpler future/queueable is needed.
- **Callout need:** Does the logic include web service callouts? If yes, prefer future (`callout=true`), queueable, or batch with `Database.AllowCallouts`.
- **Dependency on completion:** Does subsequent logic depend on this operation finishing? If async, plan a completion signal (callback, Platform Event, queueable finalizer).
- **Error handling and retry strategy:** Async errors don't surface to users — plan for logging, compensating actions, or retry queues.

## Pattern Selection Guide

| Need                                              | Pattern                |
| ------------------------------------------------- | ---------------------- |
| Simple, quick background task or callout          | `@future`              |
| Complex async with non-primitive data or chaining | Queueable              |
| Millions of records in chunks                     | Batch Apex             |
| Time-based recurring execution                    | Schedulable            |
| Real-time, returns value to caller                | Synchronous Apex class |

## Apex Class Types

### Synchronous Apex Classes

Regular Apex classes run synchronously as part of the current transaction. Use for structured code (helper, service, domain classes) invoked from triggers, LWC, or other Apex. Subject to tighter governor limits (100 SOQL, 6 MB heap, 10s CPU) but allows immediate execution and return values.

### Future Methods (`@future`)

Annotate static methods with `@future` for fire-and-forget async execution. Best for simple non-urgent tasks and external callouts.

**Limitations:**

- Only primitive types or collections of primitives as parameters (no sObjects or custom types).
- Cannot chain future methods or control execution order.
- No Job ID for monitoring.
- Max **50 future + queueable calls combined** per transaction.

```apex
@future(callout=true)
public static void callExternalService(Set<Id> recordIds) {
    // callout logic here — runs async after the calling transaction commits
}
```

### Queueable Apex (`System.enqueueJob`)

Recommended over future methods for most new async use cases. Supports instance variables (including sObjects and complex types), returns a Job ID, and supports chaining.

**Use when:**

- You need to pass non-primitive types (sObjects, custom classes) to the job.
- You need a Job ID to monitor status via `AsyncApexJob`.
- You need to chain jobs sequentially (one job triggers the next).

**Limitations:**

- Max **50 enqueues per transaction** (shared with future calls).
- Can enqueue only **one child job** from within a running job — no fan-out.
- Chaining depth default: **5 levels**.
- No automatic retry on failure.

```apex
public class MyQueueable implements Queueable {
    private List<SObject> records;

    public MyQueueable(List<SObject> records) {
        this.records = records;
    }

    public void execute(QueueableContext context) {
        // bulk-safe processing logic — no SOQL/DML in loops
        // to chain: System.enqueueJob(new NextQueueable(results));
    }
}
// Invoke from trigger/service: System.enqueueJob(new MyQueueable(triggerRecords));
```

### Batch Apex (`Database.Batchable`)

Processes very large record sets in controlled chunks. Each chunk (up to 2000 records, developer-configurable lower) runs with fresh governor limits.

**Use when:**

- Processing thousands to millions of records where a single transaction would exceed limits.
- Need stateful accumulation across chunks (`Database.Stateful`).
- Resilience against per-transaction governor limits is required.

**Limitations:**

- Max **5 batch jobs** running or queued at once; additional jobs held in Apex Flex Queue (max 100 waiting).
- Only one batch `start()` runs at a time per org — batches start sequentially.
- Requires three methods: `start`, `execute`, `finish`.
- Not suitable for near-real-time needs or trivial single-record tasks.

```apex
global class MyBatch implements Database.Batchable<SObject>, Database.Stateful {
    global Integer processedCount = 0;

    global Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator('SELECT Id, Name FROM Account WHERE IsActive__c = true');
    }

    global void execute(Database.BatchableContext bc, List<Account> scope) {
        processedCount += scope.size();
        // bulk-safe processing — no SOQL/DML in loops
        update scope;
    }

    global void finish(Database.BatchableContext bc) {
        // post-processing; kick off next job if needed
        // System.enqueueJob(new FollowUpQueueable(processedCount));
    }
}
// Invoke: Database.executeBatch(new MyBatch(), 200);
```

### Schedulable Apex (`Schedulable`)

Runs code at specified CRON-based times. Typically used to launch Batch or Queueable jobs on a schedule rather than doing heavy processing directly.

**Limitations:**

- Max **100 scheduled jobs** per org (5 in Developer Edition).
- Only one instance runs at a time; overlapping invocations are skipped.
- Minimum schedule granularity: every 1 hour (15 min with multiple schedules).
- Does not return results; log or use Platform Events to track outcomes.

```apex
global class MyScheduledJob implements Schedulable {
    global void execute(SchedulableContext ctx) {
        Database.executeBatch(new MyBatch(), 200);
    }
}
// Schedule via Apex: System.schedule('Nightly Batch', '0 0 2 * * ?', new MyScheduledJob());
// Unschedule: System.abortJob(jobId);
```

## Guardrails

**Bulkification (mandatory):**

- All methods must handle bulk input using List/Set/Map collection types.
- Never issue queries or DML for individual records inside a loop.

**No SOQL/DML in loops:**

- Collect IDs/records in a Set/Map first, query once with `IN :ids`, then process the map results.
- Prevents exceeding per-transaction limits (100 SOQL, 150 DML statements).

**Avoid recursive async calls:**

- Future methods cannot call another future or batch — runtime exception will occur.
- Batch `execute` cannot enqueue a future (same restriction).
- Queueable can enqueue exactly one child; always define a clear termination condition to prevent infinite chains.

**Async invocation limits:**

- Max **50 future + queueable** enqueues per transaction.
- For processing 1000 records, use one Batch job (1 async invocation) — not 1000 future calls.

**Queueable chaining:**

- Enqueue only one child job per running queueable execution.
- Default chaining depth: 5 levels. Contact Salesforce support to increase for specific orgs.

**Batch concurrency:**

- Only 5 batch jobs can be queued or active simultaneously.
- Stagger batch submissions; use the Apex Flex Queue intentionally to sequence jobs.

**Scheduled job limits:**

- Max 100 scheduled jobs at one time. Stagger schedules to avoid concurrency spikes.

**Governor limits in async context:**

- Async offers higher limits (12 MB heap vs 6 MB sync; 200 SOQL vs 100 sync) but is not unlimited.
- Always design for worst-case data sizes, even in async mode.

**Transaction finality:**

- If the calling transaction rolls back (e.g., trigger exception), all jobs enqueued in that transaction are also rolled back and will not execute.
- Enqueue jobs only after successful commits when order-of-operation matters.

**Security and sharing:**

- Async classes run in system mode by default — sharing rules are not enforced unless you declare `with sharing`.
- Explicitly declare `with sharing` or `without sharing` based on whether the background job should respect record-level access. Document your choice.

**Testing async Apex:**

- Wrap all async invocations in `Test.startTest()` / `Test.stopTest()`.
- Salesforce executes async jobs synchronously at `Test.stopTest()`, making results available for assertions.

## Pros and Cons

| Approach         | Pros                                                                 | Cons                                                        |
| ---------------- | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| Synchronous Apex | Immediate execution; returns values; straightforward testing         | Strict governor limits; user waits; no built-in retry       |
| `@future`        | Simple annotation; supports callouts; separates mixed-DML            | Primitives-only params; no Job ID; no chaining; 50/tx limit |
| Queueable        | Non-primitive params; Job ID; job chaining; higher limits            | 50/tx limit; single chain only; no auto-retry               |
| Batch Apex       | Handles millions of records; fresh limits per chunk; stateful option | Complex structure; slower overall; 5-job concurrency limit  |
| Schedulable      | Time-based automation; launches other jobs; simple to implement      | No return value; 100-job limit; no sub-hour precision       |

## Expected Outputs

Depending on the chosen pattern, produce the following files:

1. **Apex class (`.cls`)** — primary class implementing business logic or the async interface (`Queueable`, `Database.Batchable`, `Schedulable`).
2. **Test class (`.cls`)** — test methods with `Test.startTest()`/`Test.stopTest()` wrapping all async invocations.
3. **Invocation point** — trigger, service, or controller code calling `System.enqueueJob`, `Database.executeBatch`, or `System.schedule`.
4. **Named Credential / Remote Site Setting** — required if the class performs HTTP callouts.
5. **Logging mechanism (optional)** — Custom object, Platform Event, or email notification to surface async errors that won't appear in the UI.

## Minimum Test Suite

| Test Case                       | What to Verify                                                            |
| ------------------------------- | ------------------------------------------------------------------------- |
| Single-record / happy path      | Correct output for a simple valid input                                   |
| Bulk path (200 records)         | No SOQL/DML in loops; correct results for full collection                 |
| Async execution                 | Wrap in `Test.startTest()`/`Test.stopTest()`; assert post-execution state |
| Null / empty input              | Graceful handling of empty collections and null parameters                |
| Chaining (if used)              | Subsequent job is enqueued and produces the correct downstream result     |
| Scheduling (if used)            | `System.schedule` fires and invokes the expected logic                    |
| Toggle disabled (if applicable) | Custom setting off; no side effects occur                                 |

## Limitations to Design Around

- **No cancellation:** Async jobs cannot be canceled once enqueued. Build abort logic using a custom setting status flag checked at the start of `execute`.
- **Non-deterministic order:** Async jobs run as resources permit — order is not strictly guaranteed. Use chaining or `finish()` callbacks when sequence matters.
- **Mixed DML still applies:** Setup vs non-setup object DML restrictions still exist within each async chunk.
- **Chaining depth:** Default 5 levels for Queueable. Redesign processes that need deeper chaining (merge steps or use Batch `finish` to continue).
- **Resource throttling:** Salesforce may delay async jobs during peak platform load — jobs are not real-time.
- **24-hour rolling limit:** 250,000 combined async executions per 24 hours (or 200 per license, whichever is higher). Monitor via `OrgLimits` API for high-volume orgs.

## Definition of Done

- [ ] Correct pattern chosen for use case (Batch for large volume; not repeated future calls).
- [ ] All methods handle bulk input — no SOQL/DML inside loops.
- [ ] Async enqueue count stays within 50 per transaction.
- [ ] `with sharing` / `without sharing` declared explicitly with the reason documented in a comment.
- [ ] Error handling logs or compensates for async failures (not silently dropped).
- [ ] Jobs are idempotent — safe to re-run without causing duplicate or inconsistent data.
- [ ] Tests use `Test.startTest()`/`Test.stopTest()`, cover bulk, error, async execution, and null input paths.
- [ ] Code comments explain why async was chosen and any special configuration required.

## Reference Links

- [Async Apex overview](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_async_overview.htm)
- [Future methods](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_invoking_future_methods.htm)
- [Queueable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm)
- [Batch Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_batch_interface.htm)
- [Schedulable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_scheduler.htm)
- [Governor limits quick reference](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_apexgov.htm)
