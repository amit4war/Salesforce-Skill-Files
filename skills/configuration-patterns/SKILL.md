---
name: configuration-patterns
description: "Implement secure, deployable, metadata-driven configuration patterns for Apex, Flow, LWC, and integrations without hardcoded values. Use when introducing a feature toggle, threshold, routing value, integration endpoint, auth credential, or bypass override for triggers and flows."
argument-hint: "<config use case> <storage candidate: CMT|HCS|NamedCredential|CustomLabel> <sensitivity: secret|non-secret>"
---

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

## When Not To Use

- The value is a secret or credential — use Named Credential or Org Secret, not Custom Metadata.
- The configuration is user-generated data, not a developer-controlled setting.
- The value is a one-time migration constant used only in a script — do not create persistent metadata for it.

---

## Required Inputs

- Configuration use case and owning team/domain
- Scope: org-wide, profile/permission-based, or user-context (hierarchy) behavior
- Storage choice (see selection guide below)
- Default value and fallback behavior when config is absent
- Whether the value is environment-specific (sandbox vs. UAT vs. production)
- Whether the value is sensitive (secret, credential, or PII-adjacent)

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent default values, storage types, or field names.

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

---

## Start Here

1. Read `README.md`.
2. Load `context/`.
3. Apply `rules/`.
4. Follow `workflows/execution-steps.md`.
5. Validate with `quality/validation-checklist.md`.

## Mandatory Output Contract

- Storage selection rationale
- Metadata/provider changes
- Security handling for secrets and permissions
- Tests for configured and missing values
- Deployment and rollback notes

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — config use case, storage type chosen, sensitivity level
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — metadata definition, provider class, call-site usage
4. **Test Evidence** — configured value path and missing-config fallback path both covered
5. **Deployment and Rollback** — deploy order, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section. Never invent config values or IDs.

### Security
