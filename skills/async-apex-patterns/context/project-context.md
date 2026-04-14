# Project Context

This skill covers asynchronous business processing in Salesforce Apex.

## In Scope

- `@future`
- Queueable Apex
- Batch Apex
- Schedulable Apex
- Async invocation from triggers, services, controllers, and schedulers

## Key Assumptions

- Async is used to improve scalability, separation, or user experience.
- Jobs must be safe under retries, delayed execution, and partial operational failures.
- Org uses source control and test-driven deployment.
