# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: New Selector Method

**Prompt:** "Write a selector for Opportunity that returns open opportunities by owner, with only the fields needed for a summary view."

**Expected skill trigger:** soql-and-selectors

**Expected output shape:**
- `OpportunitySelector` class with `with sharing` declared
- Method: `findOpenByOwnerIds(Set<Id> ownerIds)` — bulk-safe signature
- SOQL: SELECT fields in alphabetical order, `WHERE` with bind variable (never concatenation), `LIMIT` present
- Security: `Security.stripInaccessible(AccessType.READABLE, results)` applied
- Unit test covering positive (records found), empty (no match), and security scenarios

**Key edge case:** Validates alphabetical field order, bind variable, LIMIT, and `stripInaccessible` preference.

---

## Golden Prompt 2: Refactor Inline SOQL

**Prompt:** "Refactor duplicated Opportunity queries scattered across `OpportunityService` and `OpportunityDomain` into `OpportunitySelector`. Update all call sites."

**Expected skill trigger:** soql-and-selectors

**Expected output shape:**
- New or updated `OpportunitySelector` with consolidated methods
- Updated call sites in `OpportunityService` and `OpportunityDomain` (no inline SOQL remaining)
- Call-site migration notes in the response
- Tests for each new selector method

**Key edge case:** Validates that no inline SOQL remains in service or domain classes after refactor.

---

## Golden Prompt 3: Halt on Missing Security Context

**Prompt:** "Write a Case selector." (No filter conditions or security context provided)

**Expected skill trigger:** soql-and-selectors

**Expected output shape:**
- Agent halts and asks: what filter conditions are needed? what security context (user-mode or system-mode)?
- No code generated until inputs are confirmed

**Key edge case:** Validates the missing-input protocol.
