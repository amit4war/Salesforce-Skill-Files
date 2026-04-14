---
name: apex-trigger-pattern
description: "Create or update Salesforce object automation using Enterprise Patterns: thin trigger, trigger handler, domain, service, and selector layers. Use when a business rule must run on insert/update/delete/undelete, refactoring existing trigger logic into enterprise layers, or standardizing trigger structure across objects for multi-developer teams."
argument-hint: "<SObject API name> <trigger events: insert|update|delete|undelete> <business rule summary>"
---

# SKILL: Apex Enterprise Trigger Pattern

## Purpose

- Create or update object automation using Salesforce Enterprise Patterns: thin trigger, trigger handler, domain, service, and selector layers.
- Production intent (why this pattern exists): This pattern keeps triggers predictable, bulk-safe, and maintainable by moving logic out of the trigger body and into structured layers that can be tested and extended as automation density grows.

## Use When

- A business rule must run on insert, update, delete, or undelete.
- Existing trigger logic needs to be refactored into Enterprise Pattern layers.
- Multiple developers need consistent trigger structure across objects.
- Spring '26 decision lens (Flow vs Apex): Use automation density (quantity, volume, dependency sprawl) to decide whether to remain declarative (Flow-first) or shift to a code-based framework for governance and performance.

## Required Inputs

- Object API name
- Trigger events in scope
- Business rules and field-change conditions
- Related domain, selector, service, or utility classes to reuse

## Additional Production-Grade Inputs

Mandatory for "LLM builds prod-grade":

- Existing automation inventory on the object (Flows, validations, legacy automation, triggers) to assess automation density risk.
- Data volume expectations (UI vs API vs bulk loads) because triggers are bulk by default and must handle record sets.
- Downstream dependency map (what other objects get created/updated/deleted) to control recursion/save-order re-entry.
- Security requirements (sharing + CRUD/FLS constraints and who executes the automation) because Apex needs explicit security consideration.
- Integration behavior if applicable (async strategy + authentication approach) because outbound message session IDs are removed and OAuth is required.

## Guardrails

- One trigger per object
- Trigger stays thin and delegates logic
- Trigger should build a shared context object through a common interface
- Trigger handler routes events and coordinates domain/service entry points
- Domain classes own object-level behavior and invariants
- Service classes orchestrate use cases and transactions
- Selector classes own reusable query access
- Use a handler factory or `getInstance()` pattern for testable handler construction
- Add a trigger on/off control using hierarchy custom settings when runtime admin control is required
- No SOQL or DML in loops
- Bulk-safe for up to 200 records
- Security-sensitive data access must be explicit
- Prefer dependency boundaries (interfaces/factories) for testable layers
- Tests must cover happy, edge, and bulk scenarios

### A) Bulkification Rules (Official)

- Triggers are optimized to operate in bulk; never assume only one record is in scope.
- Minimize SOQL by preprocessing records into Sets/Maps and issuing one query using `IN :ids`.
- Minimize DML by collecting changed records and issuing DML against collections.

### B) Correct Use of Trigger Context Variables (Official)

- Use `Trigger.new`, `Trigger.old`, `Trigger.newMap`, `Trigger.oldMap`, and context booleans (`isBefore`, `isAfter`, etc.) to route logic correctly.
- Records in `Trigger.new` can be modified only in before triggers; modifying them in after triggers causes runtime exceptions.
- `Trigger.size` is per-batch; DML operations over 200 records run in batches and invoke the trigger per batch.

### C) Governor Limits & Transaction Boundaries (Official)

- Apex runs in atomic transactions; if a limit or unhandled exception occurs, the transaction rolls back.
- Governor limits are enforced to prevent runaway code in multitenant runtime; limit exceptions can't be handled like normal exceptions.
- Design patterns must be used to stay within limits (bulk processing, efficient queries, async patterns where needed).

### D) Before vs After (Correct Placement)

- Use before for defaulting/normalizing fields and `addError()` validations to block saves.
- Use after for related-record creation that requires Ids and for launching async work to protect the save transaction.

### E) Automation Density Governance (Spring '26)

- Measure automation density across quantity, record volume, and dependency sprawl, because higher density raises the risk of save-order re-entry and CPU exhaustion.
- In high density, prefer a single orchestration entry point (this pattern) and decouple non-critical work to async paths.

### F) Integration & Authentication (Spring '26 Security Change)

- Do not rely on outbound message session IDs for integrations; Salesforce removed the ability to send session IDs in outbound messages (week of Feb 23, 2026) and requires OAuth-based authentication.

## Expected Outputs

- `ObjectTrigger.trigger`
- `ITriggerContext.cls` (or shared context interface in an existing trigger framework package)
- `ObjectTriggerContext.cls` (or equivalent implementation)
- `ObjectTriggerHandler.cls`
- `TriggerControlSettings.cls` (or equivalent custom setting provider)
- `ObjectDomain.cls` updates when needed
- `ObjectService.cls` updates when needed
- `ObjectSelector.cls` updates when needed
- `*Test.cls` updates for behavior coverage

## Working Pattern

- Keep trigger body minimal.
- Build a common trigger context instance via `getInstance()` and pass it to the handler.
- Check custom setting toggle early and exit when automation is disabled.
- Route each event to a handler override.
- Keep record-level rules in domain methods.
- Orchestrate process use cases in service methods.
- Centralize repeated query logic in selectors.
- Add bulk and field-change-aware tests.

### Production-Grade Expansion (What the LLM Must Actually Do)

- Treat every handler method as bulk-first and avoid SOQL/DML inside loops.
- Use field-change detection (`oldMap` vs `newMap`) to run expensive logic only when necessary.
- For high automation density objects, move non-critical actions to async/event-driven patterns to reduce transaction time and recursion risk.

## Reference Pattern

### Common Context Interface Contract

- `ITriggerContext` exposes operation flags (`isBefore`, `isAfter`, `isInsert`, `isUpdate`, `isDelete`, `isUndelete`) and record collections (`newList`, `oldList`, `newMap`, `oldMap`).
- `ObjectTriggerContext.getInstance()` returns a context built from Trigger static variables.

### Trigger Entry Structure

- Trigger constructs context via `ObjectTriggerContext.getInstance()`.
- Trigger calls `TriggerControlSettings.isEnabled('ObjectApiName')` and returns early when false.
- Trigger resolves handler via `ObjectTriggerHandler.getInstance(context)` and executes routing.

### Custom Setting Toggle

- Use the canonical Hierarchy Custom Setting `BypassSettings__c` with the global field `Bypass_All_Triggers__c` (disables all triggers org-wide) and per-object fields `Bypass_<ObjectApiName>_Trigger__c` (e.g., `Bypass_Account_Trigger__c`).
- Encapsulate setting access in one provider (`TriggerControlSettings`) with safe defaults (never null).
- Tests must cover both enabled and disabled paths.

> **Canonical name:** Always use `BypassSettings__c`. Never use `Trigger_Settings__c`, `TriggerBypass__c`, or `TriggerControl__c`. See `docs/ai/bypass-mechanism.md` for the full schema.

## Implementation Templates

### 1. Trigger Routing Template (Logic-Less Trigger)

Uses official trigger operation enum routing and context variables correctly. Assumes bulk processing and avoids putting logic directly in the trigger.

```apex
trigger ObjectTrigger on Object__c (
    before insert, before update, before delete,
    after insert, after update, after delete, after undelete
) {
    ITriggerContext ctx = ObjectTriggerContext.getInstance();
    if (!TriggerControlSettings.isEnabled('Object__c')) return;

    ObjectTriggerHandler handler = ObjectTriggerHandler.getInstance(ctx);
    handler.run();
}
```

---

# Apex Trigger Pattern Skill

Use this skill to deliver production-ready Apex trigger implementations that are predictable, bulk-safe, testable, and maintainable.

## Start Here

1. Read `README.md` for scope, quick start, and expected deliverables.
2. Load context from:
   - `context/project-context.md`
   - `context/tech-stack.md`
3. Apply guardrails from:
   - `rules/do-dont.md`
   - `rules/authoring-rules.md`
4. Execute the build flow in `workflows/execution-steps.md`.
5. Pick a task path from `workflows/task-playbooks.md`.
6. Validate output using:
   - `quality/validation-checklist.md`
   - `quality/common-failures.md`

## Required Inputs

- Object API name and trigger events in scope
- Business rules and field-change conditions
- Existing automation inventory (Flow, triggers, validation rules, async jobs)
- Security and permission assumptions (sharing mode, CRUD/FLS, profile/permset constraints)
- Deployment target and rollback expectations

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent values, placeholder names, or sample field names.

## Mandatory Output Contract

Every implementation produced with this skill must include:

- Thin trigger routing only
- Handler orchestration logic
- Selector/service/domain updates as needed
- Trigger toggle strategy (custom setting or metadata-driven)
- Unit tests covering happy path, bulk path, field-change path, recursion path, and toggle-off path

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — object/component name, events in scope, business rules as numbered acceptance statements
2. **Files Changed** — full relative path per file (`force-app/main/default/…`), type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — one sub-section per layer changed; what changed, why, key decisions
4. **Test Evidence** — scenarios covered, assert calls used, coverage estimate
5. **Deployment and Rollback** — deploy order, `sf` CLI command(s) to validate, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section. Never invent field names, IDs, or values.

For concrete examples, see `examples/example-prompts.md` and `examples/example-outputs.md`.
No avoidable governor-limit or security anti-patterns are introduced.
