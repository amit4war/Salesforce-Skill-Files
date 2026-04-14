# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: Replace Hardcoded Value with Custom Metadata

**Prompt:** "Replace the hardcoded invoice threshold value in `InvoiceService` with deployable Custom Metadata and a shared provider class."

**Expected skill trigger:** configuration-patterns

**Expected output shape:**
- Storage selection rationale: Custom Metadata (deployable, not a secret)
- New CMT object: `InvoiceSettings__mdt` with `InvoiceThreshold__c` field
- Provider class: `InvoiceSettingsProvider` with caching and `null` fallback (not silent null)
- `InvoiceService` updated to call provider
- Tests: configured value path AND missing-config fallback path both covered

**Key edge case:** Validates that Custom Metadata is chosen (not hardcoded), and that missing config is tested explicitly.

---

## Golden Prompt 2: Bypass Toggle for Sandbox Loads

**Prompt:** "Add a trigger bypass control for sandbox data loads and tests."

**Expected skill trigger:** configuration-patterns

**Expected output shape:**
- Storage: Hierarchy Custom Setting `BypassSettings__c` (canonical name from bypass-mechanism.md)
- Field: `Bypass_All_Triggers__c` (global) + per-object field pattern if needed
- Trigger updated to check `BypassSettings__c.getInstance()` as first action
- Tests: both bypass-on and bypass-off paths covered

**Key edge case:** Validates that `BypassSettings__c` (not `Trigger_Settings__c`) is generated.

---

## Golden Prompt 3: Halt on Missing Sensitivity Level

**Prompt:** "Store the API key for the billing system in Salesforce."

**Expected skill trigger:** configuration-patterns

**Expected output shape:**
- Agent asks: is this a secret/credential? If yes → Named Credential or Org Secret; NOT Custom Metadata
- No metadata definition generated until storage type is confirmed

**Key edge case:** Validates that secrets are never stored in Custom Metadata.
