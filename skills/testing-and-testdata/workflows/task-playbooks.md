# Task Playbooks

## Playbook A: New Class Test Suite

1. Create baseline data factory entries.
2. Add positive/negative tests.
3. Add bulk and edge coverage.

## Playbook B: Trigger Test Upgrade

1. Add 200-record bulk scenario.
2. Add field-change and recursion behavior checks.
3. Add bypass/toggle behavior tests if supported.

## Playbook C: Async/Callout Testing

1. Wrap execution in `startTest/stopTest`.
2. Configure mocks.
3. Assert async side effects and outcomes.
