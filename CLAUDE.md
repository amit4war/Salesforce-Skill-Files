# SM Accelerator Project Instructions

Use [docs/ai/baseline-salesforce-rules.md](docs/ai/baseline-salesforce-rules.md) as the canonical policy for this repository.

## Project Intent
This repository is a shared Salesforce accelerator used by multiple developers. All generated or edited output should be reusable, source-control friendly, and aligned to the standard project structure.

## Required Behaviors
- Keep Salesforce metadata in DX source format.
- Favor trigger-handler, service, and selector patterns.
- Keep Apex bulk-safe and secure.
- Avoid SOQL or DML in loops.
- Include or update tests for behavior changes.
- Update docs when shared patterns or workflow change.

## Workflow Assumptions
- Sandbox-first development model
- Feature branches merged into `main`
- Validation and deployment executed with Salesforce CLI (`sf`)
- API version target: 62.0

## Shared Context Sources
- Repository overview: [README.md](README.md)
- Project structure: [docs/project-structure.md](docs/project-structure.md)
- Team workflow: [docs/development-workflow.md](docs/development-workflow.md)
- Architecture notes: [.ai/context/project-context.md](.ai/context/project-context.md)
- Reusable skills: [skills](skills)
