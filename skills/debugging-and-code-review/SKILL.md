---
name: debugging-and-code-review
description: Diagnose issues and review Salesforce changes through evidence-based root cause analysis, minimal fixes, regression protection, and explicit review criteria.
argument-hint: "<error message or symptom> <affected component> <reproduction steps>"
---

# Debugging and Code Review Skill

Use this skill to investigate symptoms, isolate root causes, implement targeted fixes, and review changes with confidence.

## When Not To Use

- The request is to build new functionality from scratch (not debugging an existing issue).
- The symptom is actually a requirements gap, not a code defect — raise with the product owner instead.
- The change involves a full refactor unrelated to the reported symptom — scope that separately.

## Start Here

1. Read `README.md`.
2. Collect context from `context/`.
3. Apply `rules/`.
4. Follow `workflows/execution-steps.md`.
5. Validate with `quality/validation-checklist.md`.

## Required Inputs

- Symptom, error, or regression description
- Reproduction steps
- Relevant logs, code, and metadata
- Recent changes and environment details

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent error messages, stack traces, or change history.

## Mandatory Output Contract

- Root cause statement
- Evidence summary
- Minimal fix description
- Test and regression validation
- Risk, deployment, and monitoring notes
- **Upstream Analysis**: Understand what triggers the code — user actions, scheduled jobs, integration calls, or other triggers — and verify that inputs are valid before they reach the code.

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — symptom, affected component, reproduction steps
2. **Root Cause Statement** — exact cause, evidence cited
3. **Minimal Fix** — targeted change only; no unrelated refactors
4. **Test Evidence** — regression test added or updated, scenarios validated
5. **Deployment and Rollback** — risk note, deploy steps, monitoring plan

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.
