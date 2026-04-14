# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: High-Risk Deployment to UAT

**Prompt:** "Validate and deploy the AccountTrigger changes to our UAT sandbox. Use the safest test strategy and provide rollback steps."

**Expected skill trigger:** deploy-and-validate

**Expected output shape:**
- Target org alias confirmed before any command is generated
- Risk classification: High (trigger = high risk; UAT = step up one level → use `RunLocalTests`)
- Validate command: `sf project deploy validate --source-dir ... --target-org <alias> --test-level RunLocalTests`
- Deploy command after successful validation
- Rollback steps: toggle bypass, revert package, rerun impacted tests

**Key edge case:** Validates test level step-up for UAT and that validation runs before deployment.

---

## Golden Prompt 2: Scoped Manifest Validation

**Prompt:** "Create a manifest for changed files in `force-app/main/default/classes` and run a validation with specified tests only."

**Expected skill trigger:** deploy-and-validate

**Expected output shape:**
- `sf project generate manifest --source-dir force-app/main/default/classes`
- Explicit `--tests` list provided by caller (agent asks if not provided)
- `--test-level RunSpecifiedTests` flag on validate command
- Risk classification: Medium
- Warning if test list is incomplete or uncertain — recommend `RunLocalTests` instead

**Key edge case:** Validates that `RunSpecifiedTests` requires an explicit test list; agent must ask if not provided.

---

## Golden Prompt 3: Halt on Missing Org Alias

**Prompt:** "Deploy the trigger to production."

**Expected skill trigger:** deploy-and-validate

**Expected output shape:**
- Agent halts and asks: what is the target org alias? what changed components should be included?
- No `sf` command generated until alias and scope are confirmed
- Note: `NoTestRun` is not allowed in production; minimum `RunLocalTests`

**Key edge case:** Validates the missing-input protocol and production NoTestRun prohibition.
