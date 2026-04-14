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

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent values, placeholder names, or sample field names.

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
        // REQUIRED: attach a Finalizer to handle success and failure states (SonarQube High severity)
        System.attachFinalizer(new MyQueueableFinalizer());
        // bulk-safe processing logic — no SOQL/DML in loops
        // to chain: System.enqueueJob(new NextQueueable(results));
    }
}

public class MyQueueableFinalizer implements System.Finalizer {
    public void execute(System.FinalizerContext ctx) {
        if (ctx.getResult() == ParentJobResult.UNHANDLED_EXCEPTION) {
            // log failure: ctx.getException().getMessage()
        }
        // log success or trigger follow-on actions
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
        // post-processing logic — send notifications, trigger chained jobs, log completion
    }
}
```

### Schedulable Apex (`System.schedule`)

Runs Apex on a recurring schedule defined by a CRON expression. Use to trigger batch jobs or other periodic maintenance.

**Use when:**

- Business requirements dictate code runs at a specific time (nightly, weekly).
- You need to schedule a Batch Apex job to run on a recurring basis.

**Limitations:**

- Max **100 scheduled jobs** at a time org-wide.
- Schedulable itself should not contain heavy logic — delegate to Batch or Queueable.
- Scheduled jobs can be managed via `System.schedule()` or Setup UI.

```apex
public class MySchedulable implements Schedulable {
    public void execute(SchedulableContext sc) {
        // Delegate to Batch or Queueable — do not put heavy logic here
        Database.executeBatch(new MyBatch(), 200);
    }
}
// Schedule from anonymous Apex or Setup:
// System.schedule('My Nightly Job', '0 0 2 * * ?', new MySchedulable());
```

---

**No SOQL/DML in loops:** Collect all records into a List/Map before performing DML or SOQL — never inside a loop. This applies inside `execute()` batches and Queueable `execute()` methods equally.

---

## Start Here

1. Read `README.md` for scope and quick start.
2. Load context from `context/`.
3. Apply guardrails from `rules/`.
4. Follow `workflows/execution-steps.md`.
5. Use the right pattern playbook in `workflows/task-playbooks.md`.
6. Validate with `quality/validation-checklist.md`.

## Mandatory Output Contract

Every async implementation must include:

- Pattern selection rationale
- Apex class and invocation point
- Error logging or retry stance
- Tests using `Test.startTest()` and `Test.stopTest()`
- Deployment and rollback notes

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — pattern selected, business operation, invocation source
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — pattern class, invocation point, error/retry approach
4. **Test Evidence** — scenarios covered, async test structure (`Test.startTest/stopTest`), coverage estimate
5. **Deployment and Rollback** — deploy order, monitoring plan, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.
