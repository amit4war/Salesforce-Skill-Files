# Bypass Mechanism — Canonical Reference

> **Single source of truth.** All triggers, flows, and AI-generated code must use the names defined here. Never use `Trigger_Settings__c`, `TriggerBypass__c`, `TriggerControl__c`, or any other synonym.

## Hierarchy Custom Setting: `BypassSettings__c`

| Field API Name                     | Type     | Purpose                                                                 |
| ---------------------------------- | -------- | ----------------------------------------------------------------------- |
| `Bypass_All_Triggers__c`           | Checkbox | Disables **all** Apex triggers org-wide when checked                    |
| `Bypass_All_Flows__c`              | Checkbox | Prevents all record-triggered flows from executing when checked         |
| `Bypass_All_Validations__c`        | Checkbox | Prevents all validation rules from executing when checked               |
| `Bypass_Object_Trigger__c`         | Checkbox | Disables triggers for objects listed in `Skipped_Objects_Trigger__c`    |
| `Bypass_Object_Flow__c`            | Checkbox | Disables flows for objects listed in `Skipped_Objects_Flow__c`          |
| `Skipped_Objects_Trigger__c`       | Text(255)| Comma-separated object API names to skip for trigger bypass             |
| `Skipped_Objects_Flow__c`          | Text(255)| Comma-separated object API names to skip for flow bypass                |

## Per-Object Bypass Fields (optional)

For granular control, add per-object checkbox fields in the format:

`Bypass_<ObjectApiName>_Trigger__c` — e.g., `Bypass_Account_Trigger__c`, `Bypass_Case_Trigger__c`

## Usage Pattern — Apex Triggers

Every trigger must check the bypass setting as its first action:

```apex
trigger AccountTrigger on Account (before insert, before update, after insert, after update) {
    BypassSettings__c settings = BypassSettings__c.getInstance();
    if (settings.Bypass_All_Triggers__c || settings.Bypass_Account_Trigger__c) return;

    // proceed with normal trigger execution
    ITriggerContext ctx = AccountTriggerContext.getInstance();
    AccountTriggerHandler.getInstance(ctx).run();
}
```

## Usage Pattern — Record-Triggered Flows

Add a Decision Node as the **first element** in every record-triggered flow:

- Get the `BypassSettings__c` instance for the current user (Get Records or formula).
- Check `Bypass_All_Flows__c` OR `Bypass_Object_Flow__c` (and object is in `Skipped_Objects_Flow__c`).
- If either is true: route to an End element immediately without executing any further logic.

## Key Rules

- Never hardcode object API names in trigger logic — read from `Skipped_Objects_Trigger__c`.
- Always use `BypassSettings__c.getInstance()` — this returns the most specific hierarchy level (user → profile → org).
- Provide a safe default of `false` (bypass off) when the Custom Setting record does not exist.
- Every trigger test class must cover both the bypass-on and bypass-off paths.

## References

- `skills/apex-trigger-pattern/SKILL.md §Custom Setting Toggle`
- `Project Guidelines Instructions.instructions.md §9 Bypass Mechanism`
