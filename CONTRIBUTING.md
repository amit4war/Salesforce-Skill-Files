# Contributing Guide

## Purpose
This repository is shared by multiple developers building reusable Salesforce accelerator assets. Follow this guide to keep development aligned, reviewable, and deployment-ready.

## Before You Start
- Read [README.md](README.md).
- Read [docs/project-structure.md](docs/project-structure.md).
- Read [docs/development-workflow.md](docs/development-workflow.md).
- Review [docs/ai/baseline-salesforce-rules.md](docs/ai/baseline-salesforce-rules.md).

## Create a Change
1. Sync from `main`.
2. Create a feature branch.
3. Make scoped changes in the approved folder structure.
4. Add or update tests.
5. Update docs for any reusable pattern or process change.

## Coding Expectations
- Keep Apex bulk-safe.
- Avoid SOQL and DML in loops.
- Enforce CRUD and FLS handling.
- Keep LWCs accessible and state-aware.
- Prefer reuse over copy-paste implementations.

## Pull Request Checklist
- [ ] Scope is clear and reviewable
- [ ] Tests added or updated
- [ ] Validation approach documented
- [ ] Documentation updated where needed
- [ ] No hardcoded environment-specific values
- [ ] Shared patterns preserved or intentionally updated

## Commit Guidance
Use concise, traceable commit messages.

Examples:
- `feat: add lead conversion service scaffold`
- `fix: bulk-safe case trigger handler update`
- `docs: standardize team development workflow`

## When Introducing a New Pattern
If a change creates a reusable pattern:
- update the relevant file under `skills/`
- update the detailed reference under `.ai/skills/` if needed
- capture the decision in `.ai/context/project-context.md` if it affects architecture or team conventions
