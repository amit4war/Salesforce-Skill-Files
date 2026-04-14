# Example Outputs

## Example A: Implementation Summary Output

### Scope

- Object: `Account`
- Events: `before update`, `after update`
- Rule 1: Prevent `Industry` change when `Status__c = 'Inactive'`
- Rule 2: Create one follow-up task when `BillingCountry` changes

### Files Changed

- `force-app/main/default/triggers/AccountTrigger.trigger`
- `force-app/main/default/classes/AccountTriggerHandler.cls`
- `force-app/main/default/classes/AccountDomain.cls`
- `force-app/main/default/classes/AccountService.cls`
- `force-app/main/default/classes/AccountSelector.cls`
- `force-app/main/default/classes/AccountTriggerHandlerTest.cls`

### Key Decisions

- Trigger remains routing-only.
- Field-change gates prevent unnecessary execution.
- Task creation is guarded to avoid duplicates on recursion.
- Trigger toggle is checked at entry.

### Validation Evidence

- Happy path test: pass
- Bulk 200 test: pass
- Field-change negative test: pass
- Toggle-disabled test: pass

### Deployment and Rollback

- Deploy order: classes -> trigger -> tests
- Rollback: disable trigger via setting, revert package, rerun impacted tests

## Example B: Compact Diff-Style Handoff

Use this format for concise handoff:

1. What changed (by layer)
2. Why it changed (business + platform rationale)
3. How it was validated (tests and edge cases)
4. Risks and mitigations
5. Next recommended checks
