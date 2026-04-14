# Apex Trigger Pattern Skill

Production-grade skill for creating or refactoring Salesforce Apex triggers using enterprise layering.

## Purpose

Use this skill to implement trigger automation with clear boundaries:

- Thin trigger (routing only)
- Handler (event coordination)
- Domain (record invariants and validations)
- Service (orchestration and transactions)
- Selector (query reuse and query discipline)

## When To Use

- A business rule must run on `before/after insert`, `update`, `delete`, or `undelete`.
- Existing trigger logic is large, brittle, or duplicated.
- Automation density on an object is high and needs consistent governance.
- Team needs one reusable trigger implementation pattern across objects.

## When Not To Use

- Requirement is simple, low-risk, and fits declarative automation better (single Flow, no complex orchestration).
- You cannot define clear object ownership or event scope.
- No test strategy is available for the requested automation.

## Quick Start

1. Confirm object, event scope, and business rules.
2. Inventory existing automation (Flow/validation rules/triggers/async).
3. Decide Flow vs Apex using automation density and risk.
4. Implement with `workflows/execution-steps.md`.
5. Run the relevant playbook in `workflows/task-playbooks.md`.
6. Validate with `quality/validation-checklist.md` before final output.

## Expected Deliverables

- Trigger file with no business logic
- Handler class with event methods
- Domain/service/selector updates as needed
- Trigger toggle strategy (hierarchy custom setting or custom metadata)
- Comprehensive tests (happy, bulk, field-change, recursion, toggle-off)
- Deployment and rollback notes

## Skill Map

- Main entry: `SKILL.md`
- Context: `context/`
- Rules: `rules/`
- Execution workflows: `workflows/`
- Prompt + output examples: `examples/`
- Validation gates: `quality/`
- References: `references/resources.md`

## Maintenance

- Keep examples aligned with current project conventions.
- Add new failure modes to `quality/common-failures.md` as incidents are discovered.
- Update references for each Salesforce seasonal release.
