# Example Prompts (Golden Set)

Use these prompts to verify consistent skill behavior after any skill or guardrail update.

## Golden Prompt 1: Wire-Based Read Component

**Prompt:** "Create an LWC that shows Account contacts using a wire adapter on the record page. Include all UI states and Jest tests."

**Expected skill trigger:** lwc-data-patterns

**Expected output shape:**
- `@wire` to a property (preferred for read-only data) with `.data` / `.error` destructuring
- All 4 UI states: `lwc:if` for loading, empty, success, and error
- SLDS-compliant layout using `lightning-*` base components
- Check for existing shared `c/` components before creating a new one
- Jest test file with mocked `@wire` adapter
- Apex `@AuraEnabled(cacheable=true)` method if custom data source is needed

**Key edge case:** Validates `lwc:if` (not `if:true`) and wire-to-property preference.

---

## Golden Prompt 2: Imperative Save Action

**Prompt:** "Add a Save button to the renewal tracking LWC that calls Apex to update a field and refreshes the data on success."

**Expected skill trigger:** lwc-data-patterns

**Expected output shape:**
- Imperative Apex call (`cacheable=false`) on button click
- `refreshApex()` called on success (not re-wire, since wire-to-property is used)
- Error state displayed on failure using `lightning-toast` or inline message
- `@AuraEnabled` Apex method tested in `*Test.cls`

**Key edge case:** Validates distinction between `@wire` (read) and imperative (write), and correct `refreshApex` usage.

---

## Golden Prompt 3: Halt on Missing Surface

**Prompt:** "Create an LWC for account management." (No target surface, data source, or user actions specified)

**Expected skill trigger:** lwc-data-patterns

**Expected output shape:**
- Agent halts and asks: what is the target surface? what data source? what user actions are needed?
- No component code generated until inputs are confirmed

**Key edge case:** Validates the missing-input protocol.
