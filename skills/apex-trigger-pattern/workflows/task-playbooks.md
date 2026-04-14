# Task Playbooks

Use the playbook that matches the request.

## Playbook A: Create New Trigger Automation

1. Confirm object + events + acceptance criteria.
2. Create thin trigger and handler skeleton.
3. Implement business rules in domain/service.
4. Add selector queries for related data.
5. Add trigger toggle support.
6. Write complete test suite.

Deliverables: new trigger package + tests + rollout notes.

## Playbook B: Refactor Existing Trigger to Enterprise Pattern

1. Extract business logic from current trigger body.
2. Move orchestration to handler/service/domain.
3. Replace direct queries with selector calls.
4. Preserve behavior with regression tests.
5. Add missing bulk and recursion safeguards.

Deliverables: refactored layered implementation + parity tests.

## Playbook C: Add a New Business Rule to Existing Trigger

1. Locate correct event and domain boundary.
2. Add field-change gating conditions.
3. Implement minimal new logic in correct layer.
4. Add focused tests (positive, negative, bulk).
5. Confirm no regressions in existing rules.

Deliverables: incremental change with targeted tests.

## Playbook D: Stabilize High-Risk Trigger

1. Identify recursion, CPU, and query hot spots.
2. Add guardrails (recursion guard, selective execution, async deferral).
3. Reduce query and DML footprint.
4. Add stress tests and failure-path coverage.

Deliverables: safer and more stable trigger behavior under load.
