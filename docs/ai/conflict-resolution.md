# Conflict Resolution — Priority Rules for Instruction Conflicts

When instructions in different files appear to contradict each other, apply the following priority
tiers (highest priority wins). If priority is unclear, **state the conflict explicitly and ask the
caller which rule should win — never silently choose one rule**.

---

## Priority Tiers (Highest → Lowest)

### Tier 1 — HARD CONSTRAINTS (never override under any circumstances)

These rules are absolute. No justification overrides them.

| Constraint                                                      | Source                                              |
| --------------------------------------------------------------- | --------------------------------------------------- |
| No SOQL or DML inside loops                                     | Project Guidelines §3, all skills                   |
| No hardcoded IDs, secrets, credentials, or org-specific values  | Project Guidelines §3, baseline-salesforce-rules §4 |
| `with sharing` or `without sharing` must be declared on every Apex class | SonarQube ApexSharingViolations 🔴 Critical |
| `@isTest(seeAllData=true)` is prohibited                         | SonarQube ApexUnitTestShouldNotUseSeeAllDataTrue 🔴 |
| No empty catch blocks                                            | SonarQube EmptyCatchBlock 🔴 Critical               |
| No DML in trigger bodies — delegate to handler                   | SonarQube AvoidLogicInTrigger 🔴 Critical           |
| All callouts must use `https://` (never `http://`)               | SonarQube ApexInsecureEndpoint 🔴 Critical          |
| Code coverage target: **≥85%** with meaningful assertions        | Project Guidelines §3 (overrides baseline 80%)      |
| Missing Input Protocol: halt and ask — never invent values       | All SKILL.md §Required Inputs                       |

---

### Tier 2 — PROJECT GUIDELINES (override baseline-salesforce-rules)

Rules defined in `Project Guidelines Instructions.instructions.md` take precedence over
`docs/ai/baseline-salesforce-rules.md` for the same topic.

| Rule                                        | Canonical Source                               |
| ------------------------------------------- | ---------------------------------------------- |
| Naming conventions (Hungarian notation, PascalCase fields, class suffixes) | Project Guidelines §2 |
| Coverage target ≥85% (not 80%)             | Project Guidelines §3                         |
| Bypass setting name: `BypassSettings__c`   | Project Guidelines §9 + `docs/ai/bypass-mechanism.md` |
| Field API names must be PascalCase          | Project Guidelines §1                         |
| SOQL keywords ALL CAPS, fields alphabetical | Project Guidelines §4                         |
| Schedulable suffix (not "Scheduleable")     | Project Guidelines §2                         |

---

### Tier 3 — DOMAIN SKILL GUARDRAILS (override general guidelines within their domain)

When inside a domain skill's execution context, the skill's guardrails refine the general rules.

| Domain              | Overriding Rule                                                                           |
| ------------------- | ----------------------------------------------------------------------------------------- |
| SOQL & Selectors    | Security enforcement order: `stripInaccessible` > `WITH SECURITY_ENFORCED` > manual checks |
| Async Apex          | Pattern selection: follow the Pattern Selection Guide table in `async-apex-patterns/SKILL.md` |
| Apex Triggers       | Bypass toggle must use `BypassSettings__c` — not any other Custom Setting                |
| Deploy & Validate   | Test level is bound to risk classification table — never downgrade without explicit approval |

---

### Tier 4 — BASELINE-SALESFORCE-RULES (fallback)

`docs/ai/baseline-salesforce-rules.md` applies to anything not covered by Tiers 1–3.

---

### Tier 5 — SOFT RECOMMENDATIONS (advisories; may be overridden with justification)

These are preferences, not mandates. Any deviation must be documented in the response.

- Field alphabetical ordering in `SELECT`
- Reuse existing shared component (`c/`) before creating a new LWC
- Prefer `Queueable` over `@future` for new async work
- Prefer `@wire` to a property over `@wire` to a function for read-only data

---

## When Instructions Conflict and Priority Is Unclear

1. State the conflict explicitly in the response (quote both instructions).
2. Identify which priority tier each instruction belongs to.
3. Apply the higher-tier rule if it is clear.
4. If still unclear, **ask the caller which rule should win before generating any artifact**.
5. Never silently choose one rule over another.

---

## References

- `docs/ai/baseline-salesforce-rules.md` — baseline engineering standards
- `.claude/rules/Project Guidelines Instructions.instructions.md` — project-specific overrides
- `docs/ai/bypass-mechanism.md` — canonical bypass setting schema
- `skills/*/SKILL.md` — domain-specific guardrails
- `.github/instructions/sonarqube-apex-rules.instructions.md` — static analysis enforcement
