# Async Apex Patterns Skill

Production-ready guidance for future, queueable, batch, and schedulable Apex.

## When To Use

- Offloading heavy work from synchronous transactions
- Executing callouts after DML-sensitive flows
- Processing high data volume in chunks
- Scheduling recurring jobs
- Separating mixed-DML or long-running operations

## When Not To Use

- Operation must return a result immediately
- Requirement is trivial and does not justify async overhead
- Ordering and state management cannot be made deterministic

## Quick Start

1. Confirm data volume, completion dependency, and callout needs.
2. Choose pattern with `workflows/task-playbooks.md`.
3. Implement idempotent logic and error handling.
4. Test execution, bulk behavior, and failure paths.
5. Validate rollout and monitoring plan.

## Expected Deliverables

- Async class and invocation point
- Tests for execution and side effects
- Monitoring/error strategy
- Deployment and rollback guidance
