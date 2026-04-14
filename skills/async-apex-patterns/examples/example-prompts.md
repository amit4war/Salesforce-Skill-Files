# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: Queueable with Callout

**Prompt:** "Create a queueable to call an external REST API after an Account is created. The endpoint is in a Named Credential. Include error logging and tests."

**Expected skill trigger:** async-apex-patterns

**Expected output shape:**
- Pattern: Queueable + `Database.AllowCallouts`
- `System.attachFinalizer()` as first line of `execute()`
- Named Credential endpoint (`callout:` prefix, never a hardcoded URL)
- `HttpCalloutMock` used in test class
- `Test.startTest()` / `Test.stopTest()` wrapping enqueue call

**Key edge case:** Validates QueueableWithoutFinalizer rule and callout-after-DML pattern.

---

## Golden Prompt 2: Nightly Batch with Schedulable Wrapper

**Prompt:** "I need to process 50,000 Contact records nightly to update a calculated field. Use the right async pattern."

**Expected skill trigger:** async-apex-patterns

**Expected output shape:**
- Pattern selection rationale: Batch (volume > queueable capacity)
- `Database.Batchable` class with `start`, `execute`, AND `finish` methods
- `Database.Stateful` if summary logging is required
- Schedulable wrapper class calling `Database.executeBatch()`
- Tests with `Test.startTest()` / `Test.stopTest()`; `Database.executeBatch` inside that block

**Key edge case:** Validates that the `finish()` method is present and Schedulable wrapper is generated.

---

## Golden Prompt 3: Async Trigger with Mixed-DML Error

**Prompt:** "My trigger on User is failing with a mixed-DML error when I try to insert a Task. Fix it."

**Expected skill trigger:** async-apex-patterns

**Expected output shape:**
- Root cause: DML on User (setup object) and Task (non-setup) in same transaction
- Solution: defer Task insert to `@future` method after trigger transaction completes
- `@future` method accepts only primitive parameters (List<Id>)
- Test covers both the trigger path and the future method path

**Key edge case:** Validates correct `@future` usage for mixed-DML isolation.
