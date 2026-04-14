---
name: soql-and-selectors
description: "Develop reusable, secure, and governor-limit-aware SOQL query logic using the Selector pattern in Apex. Use when writing new data access queries in service or trigger logic, refactoring duplicate inline SOQL into a selector class, enforcing CRUD/FLS security centrally, optimizing query performance with selective filters and limits, or adopting Apex Enterprise Patterns that separate query logic from business logic. Covers bind variables, minimal field selection, bulk Set<Id> patterns, sharing context, WITH SECURITY_ENFORCED, stripInaccessible, naming conventions, and selector unit testing."
argument-hint: "<SObject API name> <filter conditions> <security context: user|system>"
---

# SKILL: SOQL and Selector Design

## Purpose

Develop reusable, secure, and governor-limit-aware query logic using the Selector pattern in Apex. This ensures that all data retrieval is centralized, consistent, and optimized for performance and maintainability across service and domain layers. By encapsulating SOQL in selector classes, you achieve:

- **Better Reusability**: Queries can be called from multiple places without duplication.
- **Easier Maintenance**: Field sets and conditions are defined in one place, reducing drift.
- **Governance**: All queries are optimized (selective fields, proper filters, limits) and easier to review for security (FLS/CRUD) compliance.
- **Testability**: Selector methods can be unit tested independently and mocked in tests of higher-level logic.

## When Not To Use

- The query is one-off, non-reusable, and exists in a script or anonymous Apex context only.
- The data access is already centralized in an existing selector — call the existing method instead of creating a new one.
- The query is entirely within a test class's `@TestSetup` method with no production reuse.
  Use `examples/example-prompts.md` and `examples/example-outputs.md` as output references.
  - `findByIds(Set<Id> ids)` — general bulk fetch by Id.

---

## Required Inputs

- SObject API name
- Filter conditions and indexed field references
- Security context (`user` mode with sharing enforcement, or `system` with explicit justification)
- Expected data volume (informs whether `LIMIT` is required)
- Calling layer (service, domain, handler) to establish correct dependency direction

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent field names, filter values, or object names.

## Start Here

1. Read `README.md` for scope and quick-start.
2. Load context files:
   - `context/project-context.md`
   - `context/tech-stack.md`
3. Apply rules:
   - `rules/do-dont.md`
   - `rules/authoring-rules.md`
4. Execute `workflows/execution-steps.md`.
5. Validate with `quality/validation-checklist.md`.

## Mandatory Output Contract

- Selector method design with clear naming and signatures
- Security approach (`with sharing`, `stripInaccessible`, or explicit rationale)
- Performance notes (field minimization, filters, bulk input)
- Updated call-site migration notes
- Selector-focused unit test coverage

**Security Enforcement Priority (in preference order):**

1. `Security.stripInaccessible(AccessType.READABLE, records)` — preferred for field-level enforcement in all user-facing selectors.
2. `WITH SECURITY_ENFORCED` — use only when dynamic SOQL requires it; does not strip fields silently.
3. Manual `isAccessible()` / `isCreateable()` etc. — use only when `stripInaccessible` is structurally infeasible; document the reason.

Document the justification for any choice other than option 1.

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — SObject, filter conditions, security context
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — method signature, SOQL design, security enforcement choice
4. **Test Evidence** — positive/empty/edge/security scenarios covered
5. **Deployment and Rollback** — deploy order, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.
