# Deploy and Validate Skill

Production-grade operating guide for validating and deploying Salesforce metadata with Salesforce CLI.

## Purpose

- Standardize deployment execution across developers.
- Reduce deployment risk through explicit validation and rollback planning.
- Improve traceability for pull requests and release notes.

## When To Use

- You need to validate or deploy changed metadata.
- You are preparing PR evidence for QA/UAT/prod promotion.
- You are triaging failed validation or failed tests.

## When Not To Use

- Changes are not source-controlled.
- Target org and release scope are unclear.
- You cannot provide a rollback path for risky changes.

## Quick Start

1. Confirm target org alias and branch scope.
2. Classify risk and select test level.
3. Validate first, deploy second.
4. Capture outcomes and rollback notes.

## Deliverables

- Command log (validate/deploy/test)
- Risk and test-level justification
- Failure triage summary (if any)
- Rollback procedure

## Maintenance

- Keep command examples aligned with current `sf` CLI syntax.
- Update risk matrix for each release policy change.
