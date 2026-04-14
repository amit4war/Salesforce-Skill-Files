---
name: deploy-and-validate
description: Validate and deploy Salesforce metadata safely with explicit risk controls, test-level discipline, rollback plans, and reproducible CLI workflows.
argument-hint: "<target org alias> <risk level: low|medium|high|critical> <changed source path or manifest>"
---

# SKILL: Deploy and Validate with Salesforce CLI

## Purpose

Validate and deploy Salesforce metadata changes in a repeatable, team-friendly way —
using Salesforce CLI (`sf`) aligned to project standards (API v62.0, sandbox-first, feature-branch model).

## When Not To Use

- Metadata has not yet been reviewed and approved via pull request.
- Source is not in DX format under `force-app/` — convert to source format first.
- The change is experimental or exploratory — deploy to a scratch org instead of a shared sandbox.

## Use When

- Metadata is ready for sandbox validation.
- A pull request needs reproducible validation steps.
- A developer needs to triage deployment or test failures.
- Deploying hotfixes or targeted component subsets.
- Setting up or refreshing a scratch org or sandbox.
- Generating or updating a package manifest (`package.xml`).

---

## Required Inputs

- Changed source paths or manifest (`package.xml`)
- Target org alias
- Desired test level (`NoTestRun`, `RunSpecifiedTests`, `RunLocalTests`, `RunAllTestsInOrg`)
- Risk classification: low, medium, or high
- Branch name and work item reference (for PR documentation)

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it — especially target org alias and test level. Do not assume or default these values silently.

---

## Risk Classification Guide

| Risk Level | Change Type                                           | Recommended Test Level         |
| ---------- | ----------------------------------------------------- | ------------------------------ |
| Low        | Config only (fields, page layouts, permission sets)   | `NoTestRun`                    |
| Medium     | Apex class, LWC, Flow with limited scope              | `RunSpecifiedTests` (targeted) |
| High       | Trigger, service, selector, batch, cross-object logic | `RunLocalTests`                |
| Critical   | Org-wide settings, sharing rules, profiles            | `RunAllTestsInOrg`             |

- Always step up one risk level when deploying to production or a UAT sandbox.
- Never use `NoTestRun` in production.

---

## Guardrails

### Pre-Deployment

- Always validate before deploying medium or high-risk changes.
- Never deploy directly from a local machine to production — validate first.
- Confirm the target org alias before running any command — wrong alias = wrong org.
- Check for destructive changes separately; include `destructiveChanges.xml` explicitly.
- Do not use `--ignore-errors` or `--ignore-warnings` flags — fix the root cause.
- Ensure API version on all metadata matches project target (62.0).
- Review `sfdx-project.json` to confirm `sourceApiVersion` is set to `62.0`.

### Test Level Rules

- Use explicit `--test-level` on every command — never rely on CLI defaults.
- For `RunSpecifiedTests`, always list all tests that cover the changed components.
- Do not use `RunSpecifiedTests` if you are unsure of full coverage — use `RunLocalTests` instead.
- `RunAllTestsInOrg` should only be used for production deployments or full regression cycles.

### Manifest (`package.xml`) Rules

- Keep `package.xml` scoped to the current feature — do not include unrelated components.
- Always specify the correct `<version>` tag (62.0) in `package.xml`.
- Use `sf project generate manifest` to auto-generate from changed source — do not hand-edit unless necessary.
- Validate manifest completeness: missing dependencies will cause deployment failures.

### Source Format

- All metadata must be in **DX source format** (`force-app/` directory structure).
- Do not deploy metadata-format ZIPs unless explicitly required for legacy tooling.
- Use `--source-dir` for directory-scoped deploys; use `--manifest` for manifest-scoped deploys.

### Destructive Changes

- Always handle destructive changes in a **separate deployment** after the additive change succeeds.
- Use `destructiveChangesPre.xml` for pre-destructive and `destructiveChangesPost.xml` for post-destructive operations.
- Document every destructive component in the PR with business justification.
- Never delete fields, objects, or permission sets without confirming downstream impact first.

### Error Handling

- On validation failure: capture the full error output, identify root component, fix before re-running.
- On test failure: note class name, method name, error message, and stack trace in the PR.
- On partial success: identify which components succeeded and which failed — do not re-deploy successful components unnecessarily.
- Use `sf project deploy resume` with `--job-id` to resume an interrupted async deployment.

---

## Command Reference

---

name: deploy-and-validate
description: Validate and deploy Salesforce metadata safely with explicit risk controls, test-level discipline, rollback plans, and reproducible CLI workflows.

---

# Deploy and Validate Skill

Use this skill to perform reliable Salesforce deployment validation and release execution.

## Start Here

1. Read `README.md` for operating scope and risk model.
2. Load context:
   - `context/project-context.md`
   - `context/tech-stack.md`
3. Apply controls from:
   - `rules/do-dont.md`
   - `rules/authoring-rules.md`
4. Execute `workflows/execution-steps.md`.
5. Use `workflows/task-playbooks.md` for scenario-specific paths.
6. Validate output with `quality/validation-checklist.md`.

## Mandatory Output Contract

Every execution must include:

- Target org confirmation
- Deployment scope and risk class
- Validation/deploy command sequence
- Test-level rationale and impacted tests
- Failure triage notes and rollback plan

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — target org alias, branch, changed components, risk classification
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Command Sequence** — exact `sf` CLI commands with flags for validate then deploy
4. **Test Evidence** — test level rationale, impacted test classes, expected coverage
5. **Deployment and Rollback** — rollback steps, post-deploy smoke checks, job ID capture

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.

Use `examples/example-prompts.md` and `examples/example-outputs.md` as delivery references.
