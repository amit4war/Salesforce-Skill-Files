---
name: testing-and-testdata
description: "Build deterministic Apex tests and reusable test data that validate behavior, security, and scale-ready execution paths. Use when adding new Apex logic, updating trigger/service/selector/controller behavior, improving weak test coverage, or writing tests for batch, queueable, schedulable, or future methods."
argument-hint: "<class or behavior under test> <scenario type: positive|negative|bulk|async> <security context if relevant>"
---

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

## When Not To Use

- Requirement is documentation-only with no code behavior change.
- No clear behavior contract exists yet (define the contract first, then write tests).
- The code under test is a third-party managed package class with no customization.

---

## Required Inputs

- Class or method under test (fully qualified: `ClassName.methodName`)
- Behavior contract: what should happen on success, failure, and with bulk input
- Scenarios in scope: which of positive / negative / bulk / single / async / security paths are needed
- Existing factory or `@TestSetup` patterns in use on the project

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent class names, field values, or business rules.

## Start Here

1. Read `README.md` for scope and expected outcomes.
2. Load context files:
   - `context/project-context.md`
   - `context/tech-stack.md`
3. Apply rules from:
   - `rules/do-dont.md`
   - `rules/authoring-rules.md`
4. Execute `workflows/execution-steps.md`.
5. Validate with `quality/validation-checklist.md`.

## Mandatory Output Contract

Every test task output must include:

- Scenario matrix (positive/negative/bulk/single)
- Data strategy (`@TestSetup`, factory, or both)
- Assertions on outcomes and persisted side effects
- Async/mock/security test notes when applicable
- Coverage and regression summary

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — class/method under test, scenarios in scope
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — data setup, test method structure, assertion strategy
4. **Test Evidence** — scenario matrix with pass/fail notes, coverage estimate
5. **Deployment and Rollback** — N/A for pure test files unless deploy steps are needed

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.

See `examples/example-prompts.md` and `examples/example-outputs.md`.
