# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: Full Test Suite for a Service Class

**Prompt:** "Add complete tests for `AccountService.updateBillingStatus` including positive, negative, bulk, and sharing-context scenarios."

**Expected skill trigger:** testing-and-testdata

**Expected output shape:**
- `@isTest` class: `AccountServiceTest`
- `@TestSetup` with `TestDataFactory` call (no `seeAllData=true`)
- Methods: positive (status updated), negative (validation blocked), bulk (200 records), `System.runAs()` for sharing context
- Every assert includes a descriptive failure message (ApexAssertionsShouldIncludeMessage)
- Scenario matrix listed in response

**Key edge case:** Validates `System.runAs()` presence and bulk path with exactly 200 records.

---

## Golden Prompt 2: Async Test Coverage

**Prompt:** "Write tests for `InvoiceProcessingBatch` that cover the happy path and a failure where no records match the query."

**Expected skill trigger:** testing-and-testdata

**Expected output shape:**
- `Test.startTest()` / `Test.stopTest()` wrapping `Database.executeBatch()`
- Happy path: records inserted before `startTest`, assertions after `stopTest`
- Empty path: no matching records; `finish()` still executes; no exception thrown
- No `seeAllData=true`

**Key edge case:** Validates that async tests use the correct `startTest`/`stopTest` boundary.

---

## Golden Prompt 3: Halt on Missing Contract

**Prompt:** "Write tests for the billing service." (No class name or behavior specified)

**Expected skill trigger:** testing-and-testdata

**Expected output shape:**
- Agent halts and asks: what is the fully qualified class name? what is the behavior contract?
- No test code is generated until inputs are confirmed

**Key edge case:** Validates the missing-input protocol.
