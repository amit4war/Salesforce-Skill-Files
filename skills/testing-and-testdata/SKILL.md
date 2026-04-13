# SKILL: Apex Testing and Test Data

## Purpose

Create deterministic, readable Apex tests that validate business behavior and reduce regression risk —
fully aligned with Salesforce platform constraints and project coding standards.

## Use When

- Adding new Apex business logic.
- Updating trigger, service, selector, or controller behavior.
- Updating domain layer behavior or service orchestration in Enterprise Patterns.
- Improving weak or incomplete test coverage.
- Writing tests for batch, queueable, schedulable, or future methods.
- Validating callout mocks or HTTP response handling.

---

## Required Inputs

- Target class, handler, or trigger
- Functional scenarios
- Edge conditions
- Failure conditions
- Bulk behavior expectations (min 200 records for trigger tests)

---

## Guardrails

### General

- Tests must be deterministic and independent — no reliance on org data.
- Use `@TestSetup` when multiple test methods share the same base data set.
- Never use `seeAllData=true` — create all data within the test class or a `TestDataFactory`.
- All test data must be created **before** `Test.startTest()`.
- Only code under test goes inside `Test.startTest()` / `Test.stopTest()`.
- Use `System.runAs()` to test behavior under specific user profiles or permission sets.
- All test classes must use `@isTest` annotation (keeps them out of org code size limits).

### Coverage Requirements

- Minimum **85% code coverage** per class — project standard.
- Meaningful assertions required — do NOT write tests that only inflate coverage.
- Cover all 4 scenarios per class: positive, negative, bulk (≥20 records), and single record.
- Assert both the return value **and** side effects (DML results, field updates, errors thrown).

### Assertions

- Use `System.assertEquals(expected, actual, 'message')` — always include a descriptive message.
- Use `System.assertNotEquals()` where null/empty checks apply.
- Use `System.assert(condition, 'message')` for boolean conditions.
- Never assert only that no exception was thrown — assert the outcome.
- Query the database after DML to assert persisted state, not just in-memory object values.

### Naming

- Test class name: `<ClassName>Test` (e.g., `AccountTriggerHandlerTest`).
- Test method name: `test<Scenario><Condition>` (e.g., `testInsertAccountBulkSuccess`, `testUpdateNullNameFails`).
- Follow Hungarian notation for all local variables (same as production code).

### Bypass Mechanism

- Always set the bypass Custom Setting **off** in test setup so triggers and flows execute normally.
- Add a dedicated test method that sets the bypass on and asserts that logic is skipped.

### DML and SOQL in Tests

- No SOQL or DML inside loops — even in test code.
- Use `Test.startTest()` / `Test.stopTest()` to reset governor limits for the code under test.
- For async (Queueable, Batch, Schedulable), always wrap in `Test.startTest()` / `Test.stopTest()` — this forces async execution synchronously.

### Mocking and Callouts

- Use `HttpCalloutMock` interface for all HTTP callout tests.
- Use `Test.setMock(HttpCalloutMock.class, mockInstance)` before `Test.startTest()`.
- For StaticResource-based mocks, use `StaticResourceCalloutMock`.
- Never make real callouts in test context — they will fail and are not repeatable.

### Exception and Error Handling

- Write explicit negative tests: pass invalid data and assert the correct exception type and message.
- Use `try-catch` in test methods only when asserting that a specific exception IS thrown.
- Always assert the exception message or type — not just that any exception occurred.

### Sharing and Security

- Test `with sharing` enforcement by running as a user without record access and asserting 0 results.
- Test FLS enforcement by running as a profile without field access and asserting stripped fields.
- Use `System.runAs()` for all user-context-sensitive tests.

### Asynchronous Patterns

- **Batch Apex**: call `Database.executeBatch()` inside `Test.startTest()` / `Test.stopTest()`; assert final results after `stopTest()`.
- **Queueable**: enqueue inside `Test.startTest()` / `Test.stopTest()`; assert side effects after `stopTest()`.
- **Schedulable**: use `System.schedule()` inside `Test.startTest()` / `Test.stopTest()`.
- **@future**: call inside `Test.startTest()` / `Test.stopTest()`; assert after `stopTest()`.

---

## TestDataFactory Rules

- Factory class name: `TestDataFactory` (shared across project).
- Every factory method must accept a `Boolean doInsert` parameter.
- Factory methods return SObject or List<SObject>.
- Never set fields not relevant to the test scenario — keep data minimal.
- Create required lookup/parent records inside the factory before returning the child.
- Do NOT query in the factory — pass IDs as parameters where needed.

---

## Expected Outputs

- New or updated `*Test.cls` files under `force-app/main/default/classes/`.
- Updated or new `TestDataFactory.cls` entries if reusable test data patterns are needed.
- Coverage of positive, negative, bulk, and single-record paths.

---

## Working Pattern

1. Identify the behavior contract of the class under test.
2. Build minimum required data using `TestDataFactory` or `@TestSetup`.
3. Test happy path (positive), invalid input (negative), single-record, and bulk (≥20 records).
4. For async code: wrap in `Test.startTest()` / `Test.stopTest()`.
5. For callouts: register mock before `Test.startTest()`.
6. For user-context tests: wrap in `System.runAs()`.
7. Query the database after `Test.stopTest()` to assert persisted outcomes.
8. Keep assertions specific, named, and meaningful.

---

## Definition of Done

- [ ] Tests are `@isTest` annotated, in separate files, not counted toward org limits.
- [ ] ≥85% coverage with meaningful assertions — no coverage-only tests.
- [ ] Positive, negative, bulk, and single-record scenarios covered.
- [ ] All test data created within the test class — no `seeAllData=true`.
- [ ] Bypass mechanism tested (on and off).
- [ ] Async patterns wrapped in `Test.startTest()` / `Test.stopTest()`.
- [ ] Callouts use `HttpCalloutMock` — no live HTTP calls.
- [ ] `System.runAs()` used for sharing and FLS verification.
- [ ] Database state asserted after DML — not only in-memory values.
- [ ] Tests are clear, stable, and team-readable.
- [ ] All existing tests pass — no regressions introduced.
