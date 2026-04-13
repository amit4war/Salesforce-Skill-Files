---
name: apex-trigger-pattern
description: "Create or update Salesforce object automation using Enterprise Patterns: thin trigger, trigger handler, domain, service, and selector layers. Use when a business rule must run on insert/update/delete/undelete, refactoring existing trigger logic into enterprise layers, or standardizing trigger structure across objects for multi-developer teams."
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

- Use a Hierarchy Custom Setting (for example `Trigger_Settings__c`) with per-object booleans (for example `Account_Enabled__c`) or a global `Disable_All_Triggers__c` plus object-specific fields.
- Encapsulate setting access in one provider (`TriggerControlSettings`) with safe defaults.
- Tests must cover both enabled and disabled paths.

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

    // Route by operation type (cleaner than nested ifs)
    switch on Trigger.operationType {
        when BEFORE_INSERT  { handler.beforeInsert(ctx.newList); }
        when BEFORE_UPDATE  { handler.beforeUpdate(ctx.oldMap, ctx.newMap); }
        when BEFORE_DELETE  { handler.beforeDelete(ctx.oldMap); }
        when AFTER_INSERT   { handler.afterInsert(ctx.newList, ctx.newMap); }
        when AFTER_UPDATE   { handler.afterUpdate(ctx.oldMap, ctx.newMap); }
        when AFTER_DELETE   { handler.afterDelete(ctx.oldMap); }
        when AFTER_UNDELETE { handler.afterUndelete(ctx.newList, ctx.newMap); }
    }
}
2. Bulk Query Template (Selector Pattern)
Uses "collect IDs then query once with IN clause" bulk idiom.
public with sharing class ObjectSelector {
    public static Map<Id, Related__c> byParentIds(Set<Id> parentIds) {
        if (parentIds == null || parentIds.isEmpty()) return new Map<Id, Related__c>();
        return new Map<Id, Related__c>([
            SELECT Id, Parent__c, Field1__c
            FROM Related__c
            WHERE Parent__c IN :parentIds
        ]);
    }
}

3. Recursion Guard Template (Static Set)
Recursion guard is required in high dependency sprawl scenarios to prevent re-entry loops and duplicated work.
public class RecursionGuard {
    private static Set<String> keys = new Set<String>();

    public static Boolean firstRun(String key) {
        if (keys.contains(key)) return false;
        keys.add(key);
        return true;
    }
}

Definition of Done
Trigger pattern is consistent with Enterprise Pattern standards.
Shared trigger context interface and getInstance() construction are used consistently.
Trigger enable/disable behavior is configuration-driven through custom settings where required.
New logic is bulk-safe and reviewable.
Tests prove target behavior and guard against regressions.
No avoidable governor-limit or security anti-patterns are introduced.
LLM Guardrail Pack (Mandatory) ✅
Use this section as the "agent instruction set" to build production-grade triggers using the skill above.

1. LLM Operating Rules (Hard Constraints)
Do not assume single-record triggers; always implement collection-based logic.
Do not put SOQL or DML inside loops; use Set/Map preprocessing and bulk DML on lists.
Do not modify Trigger.new records in after triggers; only modify in before triggers.
Do not introduce multiple triggers per object; consolidate routing into one trigger per object.
Do not rely on Outbound Message session IDs (removed Spring '26); use OAuth-based authentication for external callbacks.
Do not implement "always-run" heavy logic; gate expensive work behind field-change checks and/or async boundaries.
2. LLM "Do / Don't" Checklist
DO ✅

Use Trigger.operationType or equivalent flags to route events cleanly.
Use Trigger.newMap.keySet() to collect IDs, query once, map results for O(1) lookups.
Implement recursion control for any follow-up DML that can cause re-entry.
Put all SOQL in selectors and all orchestration in services; keep handler focused on routing/coordination.
Add a trigger on/off switch (Hierarchy Custom Setting or Custom Metadata), check it first, and test both paths.
Write bulk tests with 200 records to validate bulk safety and batch invocation behavior.
DON'T ❌

Don't create child records in before insert if you need parent Ids (use after insert).
Don't perform "update same object" DML in before update (it causes runtime exceptions); if needed, do after update with recursion guard and selective DML.
Don't ignore automation density; high density requires tight orchestration or async decoupling to prevent CPU exhaustion.
3. Production Limitations the LLM Must Explicitly Design Around
Triggers run within Apex transactions; failures roll back the transaction.
Limits exist because of multitenant runtime; exceeding limits throws runtime exceptions that can't be "handled away."
Bulk DML > 200 records is processed in batches; triggers run once per batch and Trigger.size is per batch.
Trigger context variable constraints exist (before vs after modifications and what's available per event).
Outbound Message session ID support is removed (Spring '26); integrations must migrate to OAuth tokens.
4. Minimum Test Suite Template (Must Generate/Follow)
All implementations must include tests that cover these cases:

Happy path (single record): verifies correct behavior for UI-like operations.
Bulk path (200 records): verifies no SOQL/DML in loops, correct results for all records.
Field-change path: verifies logic runs only when monitored fields change (old vs new comparison).
Recursion path: simulates re-entry conditions and verifies recursion guard prevents duplicate/infinite work.
Toggle disabled path: sets custom setting to off and asserts no side effects.
5. Agent Output Rules (How the LLM Should Write Deliverables)
When asked to build/modify a trigger using this skill, the agent must output:

Trigger file (thin routing only)
Handler class with event methods and recursion guard
Selector class for SOQL access
Service/domain classes as needed
Trigger control provider + metadata instruction for settings
Unit tests that cover the minimum suite

Quick Links
- [Salesforce bulk trigger best practices](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_bestpract.htm)
- [Trigger context variables](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_context_variables.htm)
- [Trigger context considerations](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_context_variables_considerations.htm)
- [Automation density (Flow vs Apex) Spring '26](https://www.salesforce.com/blog/record-triggered-automation-apex-or-flow/)
- [Governor limits quick reference](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_apexgov.htm)
- [Spring '26 outbound message session ID removal](https://help.salesforce.com/s/articleView?id=005232763&language=en_US&type=1)
```
