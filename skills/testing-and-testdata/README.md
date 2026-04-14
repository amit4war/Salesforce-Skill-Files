# Testing and Test Data Skill

Production-focused guide for deterministic Apex testing and reusable data factories.

## When To Use

- New Apex logic needs tests
- Existing tests are weak or flaky
- Trigger/service/selector behavior changed
- Async or callout behavior requires validation

## When Not To Use

- Requirement is only documentation with no code behavior impact
- No clear behavior contract exists yet

## Quick Start

1. Define behavior contract and scenarios.
2. Design test data strategy.
3. Write tests across positive/negative/bulk/single paths.
4. Add async/callout/security tests where needed.
5. Validate assertions and regression impact.
