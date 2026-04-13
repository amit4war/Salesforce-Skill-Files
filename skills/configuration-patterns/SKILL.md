# SKILL: Configuration and Environment Patterns

## Purpose

Implement reusable, metadata-driven configuration so Apex, LWC, Flows, and integrations can vary
behavior by environment without code changes — keeping all settings secure, deployable, and
source-control friendly.

## Use When

- Introducing a feature toggle, threshold, routing value, or business rule setting.
- Replacing hardcoded constants, IDs, URLs, or credentials in code.
- Defining environment-specific behavior for sandbox, UAT, or production.
- Adding a new integration endpoint, auth credential, or external system reference.
- Creating org-wide, profile-based, or user-level behavioral overrides.
- Implementing or extending the bypass mechanism for triggers, flows, or validations.

---

## Required Inputs

- Configuration use case and owning team/domain
- Scope: org-wide, profile/permission-based, or user-context (hierarchy) behavior
- Storage choice (see selection guide below)
- Default value and fallback behavior when config is absent
- Whether the value is environment-specific (sandbox vs. UAT vs. production)
- Whether the value is sensitive (secret, credential, or PII-adjacent)

---

## Storage Mechanism Selection Guide

| Scenario                                                   | Storage Mechanism                              |
| ---------------------------------------------------------- | ---------------------------------------------- |
| Deployable business rules, thresholds, feature toggles     | Custom Metadata Type (CMT)                     |
| Org-wide or user/profile-overridable settings (non-secret) | Hierarchy Custom Setting                       |
| Integration endpoint URLs and auth credentials             | Named Credential                               |
| User-facing labels, messages, and UI text                  | Custom Label                                   |
| Environment-specific secrets or API keys                   | Named Credential or Org Secret (never CMT)     |
| Bypass flags for triggers, flows, and validations          | Hierarchy Custom Setting (`BypassSettings__c`) |

### Custom Metadata Type (CMT) — Preferred for business config

- Deployable via change sets and `sf` CLI — source-controllable.
- Queryable in SOQL and accessible without DML limits.
- Use for: feature flags, routing rules, threshold values, business parameters.
- API name format: `<Domain>Setting__mdt` (e.g., `CapacityRuleSetting__mdt`).
- Record developer names: PascalCase, no spaces (e.g., `Default_Capacity_Threshold`).
- Never store secrets or credentials in CMT — they are visible in source control.

### Hierarchy Custom Setting — Use for bypass and user-overridable settings

- Supports org-level, profile-level, and user-level overrides (hierarchy wins).
- Not deployable as records natively — use Custom Metadata for deployable config instead.
- Use for: bypass mechanism flags, user-preference toggles, debug mode switches.
- API name format: `<Purpose>Settings__c` (e.g., `BypassSettings__c`).
- Field API names: PascalCase, no underscores except `__c`.

### Named Credentials — Required for all integration endpoints and auth

- Always use Named Credentials to reference external endpoints — never hardcode URLs.
- Use External Credentials + Permission Sets for fine-grained callout access control.
- Reference in Apex via `Named_Credential_Name` in `HttpRequest.setEndpoint()`.
- Reference format: `callout:NamedCredentialName/path`.
- Store per-environment credentials in separate Named Credentials (e.g., `ExternalAPI_Sandbox`, `ExternalAPI_Prod`).

### Custom Labels — User-facing text only

- Use for translatable UI strings, error messages shown to users, and help text.
- Do NOT use for business logic values, thresholds, or IDs.
- Access in Apex: `System.Label.MyLabelName`.
- Access in LWC: import from `@salesforce/label/c.MyLabelName`.
- Access in Flow: use the `$Label` global variable.

---

## Guardrails

### Security

- **Never** hardcode secrets, org IDs, record IDs, endpoints, usernames, or passwords in code or CMT.
- **Never** store credentials in Custom Metadata — they are visible in source control.
- Use Named Credentials with External Credentials for all callout auth.
- Apply FLS checks before returning config values that include user-visible field data from CMT records.
- Config provider classes must be `with sharing` unless there is a documented justification.

### Design

- Centralize all configuration retrieval in a single provider/service class (e.g., `ConfigurationService`).
- Callers must never query CMT or Custom Settings directly — always go through the provider.
- Provider methods must return a safe default or throw a documented exception when config is missing — never return `null` silently.
- Use lazy loading (cached static property) in the provider to avoid repeated SOQL within the same transaction.
- Never use `instanceof` or type-switch patterns to vary behavior — externalize the variant in config.

### Bypass Mechanism (Project Standard)

- All triggers must check `BypassSettings__c.Bypass_All_Triggers__c` at the very first line.
- All record-triggered flows must check `BypassSettings__c.Bypass_All_Flows__c` as their entry criteria.
- Object-specific bypasses use `Bypass_Object_Trigger__c` / `Bypass_Object_Flow__c` with the comma-separated object list in `Skipped_Objects_Trigger__c` / `Skipped_Objects_Flow__c`.
- Test the bypass mechanism: one test method with bypass ON (assert no logic runs) and one with bypass OFF (assert logic runs).

### Naming

- CMT API names: `<Domain>Setting__mdt`
- CMT record developer names: PascalCase (e.g., `Max_Daily_Capacity`)
- Custom Setting API names: `<Purpose>Settings__c`
- Custom Setting field API names: PascalCase, no internal underscores
- Config provider class: `<Domain>ConfigurationService` or shared `ConfigurationService`
- Config provider methods: `get<SettingName>()` returning a typed value (e.g., `getMaxDailyCapacity()`)

### Source Control

- All CMT type definitions (`.object` files) and records (`.md` files) must be committed to `force-app/`.
- Custom Setting field definitions must be committed — records are org-specific and excluded.
- Named Credential definitions (without secrets) must be committed.
- Custom Labels must be committed in `customLabels/CustomLabels.labels-meta.xml`.

---

## Configuration Provider Pattern (Apex)

```apex
public with sharing class ConfigurationService {

    private static Map<String, CapacityRuleSetting__mdt> mapSettingCache;

    public static Decimal getMaxDailyCapacity() {
        CapacityRuleSetting__mdt objSetting = getSettings('Default');
        if (objSetting == null || objSetting.MaxDailyCapacity__c == null) {
            throw new ConfigurationException('MaxDailyCapacity configuration record not found.');
        }
        return objSetting.MaxDailyCapacity__c;
    }

    private static CapacityRuleSetting__mdt getSettings(String sDeveloperName) {
        if (mapSettingCache == null) {
            mapSettingCache = new Map<String, CapacityRuleSetting__mdt>();
            for (CapacityRuleSetting__mdt objRec : [
                SELECT DeveloperName, MaxDailyCapacity__c
                FROM CapacityRuleSetting__mdt
            ]) {
                mapSettingCache.put(objRec.DeveloperName, objRec);
            }
        }
        return mapSettingCache.get(sDeveloperName);
    }
}
```

---

## LWC Configuration Access

- Use `@salesforce/label` imports for Custom Labels.
- Use `@wire(getRecord)` or imperative Apex for CMT-driven values surfaced via an Apex provider.
- Never query CMT or Custom Settings directly from LWC JavaScript — always go through an Apex method.

```javascript
import MAX_CAPACITY_LABEL from "@salesforce/label/c.MaxCapacityHelpText";
```

---

## Flow Configuration Access

- Use `{!$CustomMetadata.CapacityRuleSetting__mdt.Default.MaxDailyCapacity__c}` to reference CMT values directly in flows (supported natively).
- Use `{!$Label.MyLabelName}` for Custom Labels in flows.
- Never hardcode threshold values in flow decision nodes — always reference CMT or Custom Labels.

---

## Expected Outputs

- CMT type and/or record metadata files under customMetadata
- Custom Setting field definitions under objects
- Named Credential definition files (secrets managed separately in org)
- Custom Label entries in `CustomLabels.labels-meta.xml`
- New or updated `ConfigurationService` (or domain-specific provider) class
- Tests covering: configured value, missing config (exception or default), fallback paths, bypass on/off
- Brief documentation note if a new shared configuration type is introduced

---

## Working Pattern

1. Classify the setting: business rule, UI text, integration endpoint, bypass flag, or secret.
2. Select storage mechanism using the guide above.
3. Define the CMT/Custom Setting field structure — agree naming with the team upfront.
4. Add or update the shared configuration provider — never duplicate retrieval logic.
5. Implement caching (static map) in the provider to avoid repeated SOQL.
6. Define default value and exception behavior for missing config.
7. Update callers to go through the provider — remove any direct queries or hardcoded values.
8. Write tests: config present (happy path), config absent (fallback/exception), bypass on, bypass off.
9. Commit all metadata to source control — confirm secrets are NOT checked in.

---

## Definition of Done

- [ ] No hardcoded IDs, URLs, credentials, or threshold values anywhere in code.
- [ ] Storage mechanism chosen matches the security and deployability requirements.
- [ ] All configuration retrieved through a single centralized provider class.
- [ ] Provider uses lazy-loaded static cache — no repeated SOQL per transaction.
- [ ] Missing config produces a documented exception or safe default — never silent `null`.
- [ ] Bypass mechanism fields present and checked in all triggers and record-triggered flows.
- [ ] CMT type definitions, records, Custom Setting fields, Labels, and Named Credentials committed to source control.
- [ ] Secrets and credentials stored in Named Credentials — NOT in CMT or source control.
- [ ] Tests cover: configured value, missing config, bypass on, and bypass off.
- [ ] Callers use the provider API — no scattered direct queries or hardcoded constants remain.
- [ ] New configuration types documented for team reuse.
