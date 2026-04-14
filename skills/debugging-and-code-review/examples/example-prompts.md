# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: Runtime Exception Diagnosis

**Prompt:** "Users get `List index out of bounds` on Opportunity save. Diagnose root cause and implement the minimal fix with a regression test."

**Expected skill trigger:** debugging-and-code-review

**Expected output shape:**
- Scope Confirmation: symptom, affected component, reproduction steps
- Root cause statement (exact line or pattern causing the error)
- Upstream analysis: what triggers the code path (user action, flow, trigger, API?)
- Minimal fix only — no unrelated refactors
- Regression test added
- Risk and monitoring notes

**Key edge case:** Validates upstream analysis is performed and fix is targeted, not a broad refactor.

---

## Golden Prompt 2: Bulk-Safety Code Review

**Prompt:** "Review this trigger refactor for bulk-safety, null handling, and unintended downstream changes."

**Expected skill trigger:** debugging-and-code-review

**Expected output shape:**
- Evidence checklist: SOQL in loops? DML in loops? Null checks missing? Recursion risk?
- Specific line-level callouts for each violation found
- Downstream impact analysis
- Test evidence confirming bulk path passes

**Key edge case:** Validates that review is evidence-based, not advisory prose.

---

## Golden Prompt 3: Halt on Missing Reproduction Steps

**Prompt:** "Something is broken in the Case trigger. Fix it."

**Expected skill trigger:** debugging-and-code-review

**Expected output shape:**
- Agent halts and asks: what is the symptom? what are the reproduction steps? what logs or errors are available?
- No fix generated until sufficient context is provided

**Key edge case:** Validates the missing-input protocol.
