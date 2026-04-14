# Execution Steps

Follow this sequence end-to-end for every trigger task.

## 1) Intake and Scope

1. Confirm object API name and events in scope.
2. Capture business rules as numbered acceptance statements.
3. Capture non-functional requirements (performance, security, auditability).

Expected output: scoped requirement list.

## 2) Inventory Existing Automation

1. Identify existing trigger(s), Flow(s), validation rules, and async jobs.
2. Record potential collisions and save-order risks.
3. Decide whether to extend existing framework components or create new ones.

Expected output: automation inventory with risks.

## 3) Design Layer Responsibilities

1. Define trigger routing behavior.
2. Map each business rule to handler/domain/service/selector.
3. Define field-change gates and recursion boundaries.
4. Define trigger toggle strategy (custom setting or metadata).

Expected output: layer-to-rule mapping.

## 4) Implement

1. Create/update thin trigger file.
2. Create/update handler event methods.
3. Implement domain/service logic.
4. Add or update selector queries.
5. Add/verify recursion guard where needed.

Expected output: compile-ready code changes.

## 5) Test and Validate

1. Add tests for:
   - happy path
   - bulk (200 records)
   - field-change gating
   - recursion prevention
   - toggle disabled
2. Confirm no SOQL/DML in loops.
3. Confirm behavior in both before/after contexts.

Expected output: passing tests and risk notes.

## 6) Deployment and Rollback Planning

1. Define deployment order (metadata, classes, trigger, tests).
2. Define rollback strategy:
   - toggle off automation
   - revert metadata/code package
   - revalidate impacted object behavior
3. Document post-deploy checks.

Expected output: deploy + rollback checklist.
