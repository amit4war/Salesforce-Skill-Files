# Do and Don't Guardrails

## Do

- Do reproduce before fixing.
- Do inspect upstream triggers, flows, and inputs.
- Do inspect downstream side effects and integrations.
- Do add regression tests.
- Do explain why the fix works.

## Don't

- Don't patch symptoms only.
- Don't mix unrelated refactors into a bug fix.
- Don't approve changes without test evidence.
- Don't ignore deployment monitoring for risky fixes.
