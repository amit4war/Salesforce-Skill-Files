# Example Prompts

## Prompt 1: New Trigger

"Create Account trigger automation. On before update, block industry changes for inactive accounts. On after update, when `BillingCountry` changes, create one follow-up task. Use enterprise layers and include full tests."

Expected behavior:

- Agent asks for missing constraints (task owner, due date, bypass rules).
- Agent builds thin trigger + handler + domain/service/selector as needed.
- Agent adds field-change gate and recursion protection.
- Agent provides tests for single and bulk updates.

## Prompt 2: Refactor Legacy Trigger

"Refactor `OpportunityTrigger` into enterprise pattern. Keep existing behavior exactly, remove SOQL from loops, and add toggle-based kill switch."

Expected behavior:

- Agent inventories existing logic before moving code.
- Agent preserves behavior with regression tests.
- Agent introduces toggle provider and disabled-path test.

## Prompt 3: Risk Reduction

"Our Case trigger hits CPU limits in bulk API loads. Optimize it with selective execution and async for non-critical operations."

Expected behavior:

- Agent identifies hot paths and dependency sprawl.
- Agent applies field-change checks and bulk query consolidation.
- Agent defers non-critical work to async where safe.
- Agent documents tradeoffs and post-deploy checks.
