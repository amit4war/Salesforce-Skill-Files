# Project Context

## Domain Focus

This skill targets Salesforce object automation implemented through Apex triggers and enterprise layering.

## Scope

In scope:

- Trigger creation or refactoring for one object at a time
- Event-specific automation (`before`/`after`, `insert`/`update`/`delete`/`undelete`)
- Layered architecture updates (trigger, handler, domain, service, selector)
- Test coverage and deployment readiness checks

Out of scope:

- Large-scale domain redesign across unrelated objects
- UI/LWC-only changes with no trigger impact
- Unbounded cross-org migration planning

## Assumptions

- Org uses Salesforce DX source format.
- Trigger framework conventions may already exist and should be reused first.
- One trigger per object is the baseline operating standard.

## Environment Assumptions

- Work runs in a development sandbox or scratch org before production.
- Deployment runs through source control and CI/CD gates.
- Permissions and field visibility may vary by profile and permission set.

## Decision Lens: Flow vs Apex

Choose Apex trigger pattern when:

- Multi-step orchestration is needed.
- Strict transaction control is required.
- Bulk and dependency complexity exceeds declarative maintainability.

Choose Flow when:

- Requirement is straightforward and low density.
- Declarative implementation is sufficient and easier to maintain.
