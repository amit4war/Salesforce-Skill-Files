# Project Context

## Scope

This skill governs Salesforce metadata validation and deployment operations from source-controlled project artifacts.

## Environment Assumptions

- Sandbox-first execution model.
- Feature-branch PR workflow.
- DX source format under `force-app/`.

## Deployment Scope Boundaries

In scope:

- Additive metadata deploys
- Targeted deploys by source path or manifest
- Validation-only runs
- Test-level strategy and execution

Out of scope:

- Direct production changes without validation
- Untracked manual org modifications
