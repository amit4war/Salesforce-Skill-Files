# Baseline Salesforce Development Rules

This document is the canonical engineering standard for the SM Accelerator repository. All AI assistants, developers, reviewers, and CI checks should align to these rules.

## 1. Core Principles
- Build for reuse across the org, not for one-off local success.
- Prefer simple, bulk-safe, deployable patterns over clever shortcuts.
- Keep source in Salesforce DX format and ready for CI validation.
- Design for team development: clear ownership, predictable structure, and low merge friction.
- Treat security, testability, and maintainability as release requirements.

## 2. Repository Standards
- Use a single shared repository structure for all contributors.
- Keep all deployable metadata under `force-app/main/default` unless package strategy changes.
- Keep automation, scripts, and docs in predictable locations.
- Avoid personal folders, machine-specific config, or org-specific hardcoding in committed files.
- Document every new reusable pattern in the relevant skill or project doc.

## 3. Apex Standards (Enterprise Pattern Framework)
- Use Salesforce Enterprise Patterns as the default architecture for Apex business logic.
- One trigger per object, with triggers kept thin and delegating to a `TriggerHandler`.
- Organize logic by responsibility: `Domain` for object behavior/invariants, `Service` for orchestration/use cases, `Selector` for query access, and dedicated orchestration classes where needed.
- Keep data access in selectors and keep DML orchestration in service-layer workflows (for example with unit-of-work style coordination).
- No SOQL or DML inside loops.
- Bulk-safe behavior is required for up to 200 records.
- Prefer dependency boundaries (interfaces/factories) so service and selector layers are testable and replaceable.
- Use meaningful class names ending with `Trigger`, `TriggerHandler`, `Service`, `Selector`, `Domain`, or `Test` where applicable.
- Keep methods focused, deterministic, and easy to test.

## 4. Security Standards
- Enforce CRUD and FLS with `WITH SECURITY_ENFORCED`, `Security.stripInaccessible()`, or user-mode data operations as appropriate.
- Use bind variables in SOQL.
- Do not hardcode secrets, session identifiers, org IDs, or integration credentials.
- Use Named Credentials or metadata-driven configuration for integrations.
- Avoid exposing sensitive information in debug logs, exceptions, or UI messages.

## 5. SOQL and Data Access Standards
- Query only required fields.
- Add `LIMIT` unless intentionally unbounded and clearly justified.
- Centralize reusable access patterns in selector classes.
- Keep filtering explicit and indexed where possible.
- Use aggregate queries or maps/sets when they simplify governor-safe processing.

## 6. LWC Standards
- Prefer SLDS 2
- Prefer `@wire` for read use cases and imperative Apex for explicit user actions.
- Always handle loading, empty, success, and error states.
- Use SLDS and accessible markup.
- Keep business logic out of client controllers when it belongs in Apex.
- Surface user-friendly error messages; log technical details only when appropriate.

## 7. Testing Standards
- Create or Use TestDataFactory
- Add or update tests for every functional change.
- Tests must be deterministic, isolated, and readable.
- Use `@TestSetup` where shared setup improves clarity.
- Cover happy path, edge path, error path, and bulk behavior when relevant.
- Assert outcomes and side effects, not only execution.
- Target at least 80% coverage on changed Apex classes, with meaningful assertions.

## 8. Metadata Standards
- Keep metadata source-control friendly and deployable.
- Include descriptions and help text for new fields where appropriate.
- Keep naming consistent across objects, fields, classes, flows, and LWCs.
- Prefer permission sets over profile-heavy customization.
- Avoid drift caused by manual sandbox-only configuration.

## 9. Team Workflow Standards
- Follow feature branches with pull requests into `main`.
- Keep PRs scoped, reviewable, and linked to business intent.
- Validate deployable changes before merge.
- Document assumptions, risks, and follow-up items in the PR.
- Resolve conflicts in source, not by reauthoring generated artifacts unless necessary.

## 10. Documentation Standards
- Update `README.md` when project setup, workflow, or structure changes.
- Update `.ai` skill files when a reusable development pattern evolves.
- Record architectural decisions in `.ai/context/project-context.md`.
- Prefer templates and examples that the whole team can reuse.

## 11. Definition of Done
A change is complete only when:
1. Source is committed in the agreed folder structure.
2. Security and governor-limit considerations are addressed.
3. Tests are added or updated.
4. Deployment validation steps are identified or executed.
5. Documentation is updated for any reusable behavior, workflow, or structure change.
