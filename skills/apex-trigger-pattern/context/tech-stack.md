# Tech Stack and Constraints

## Core Stack

- Salesforce Platform
- Apex triggers and classes
- SFDX source structure
- Apex test framework

## Architecture Constraints

- One trigger per object.
- Trigger must remain thin (routing only).
- Business logic must live in handler/domain/service layers.
- SOQL access must be centralized in selectors where feasible.

## Runtime Constraints

- Triggers are bulk by default.
- Governor limits apply per transaction.
- Transaction failure rolls back all changes.

## Security Constraints

- Enforce explicit sharing model decisions.
- Respect CRUD/FLS boundaries in data operations.
- Avoid exposing sensitive values in logs or exception messages.

## Integration Constraints

- Use OAuth-based authentication for integrations.
- Do not rely on outbound message session IDs.
- Prefer async patterns for non-critical external side effects.

## Deployment Constraints

- Deploy with tests and validate impacted automation paths.
- Include rollback plan before production deployment.
