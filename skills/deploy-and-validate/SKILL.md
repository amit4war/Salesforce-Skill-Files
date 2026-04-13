# SKILL: Deploy and Validate with Salesforce CLI

## Purpose

Validate and deploy Salesforce metadata changes in a repeatable, team-friendly way —
using Salesforce CLI (`sf`) aligned to project standards (API v62.0, sandbox-first, feature-branch model).

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

### Generate a manifest from source

```bash
sf project generate manifest --source-dir force-app --name manifest/package.xml
```
