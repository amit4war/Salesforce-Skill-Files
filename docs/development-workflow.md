# Development Workflow

This workflow is the default team model for SM Accelerator.

## Delivery Model
- Development model: sandbox-first
- Source control model: feature branches with pull request merge into `main`
- Deployment interface: Salesforce CLI (`sf`)
- API version target: 62.0

## Branching Standard
- `main`: production-ready or integration-ready baseline
- `feature/<work-item>-<short-description>`: developer feature branch
- `hotfix/<work-item>-<short-description>`: urgent production or release fix

## Recommended Daily Flow
1. Pull latest `main`.
2. Create a feature branch.
3. Develop in the shared repository structure.
4. Run focused validation and tests locally.
5. Update documentation if patterns or structure changed.
6. Raise a pull request with deployment notes and risk summary.
7. Merge after review and successful validation.

## Pull Request Expectations
- Clear summary of the business or technical objective
- List of metadata or modules changed
- Test evidence or validation command output summary
- Risks, assumptions, and follow-up items
- Screenshots for UI changes when relevant

## Validation Guidance
### Small changes
- Run targeted tests where possible.
- Validate only changed paths when appropriate.

### Medium or high-risk changes
- Validate before merge.
- Prefer `RunLocalTests` or targeted suites based on impacted domains.
- Record failure details and remediation steps in the PR.

## Command Reference
### Validate a changed source directory
`sf project deploy validate --source-dir force-app --test-level RunLocalTests --target-org <alias>`

### Deploy after successful validation
`sf project deploy start --source-dir force-app --test-level RunLocalTests --target-org <alias>`

### Run Apex tests
`sf apex run test --target-org <alias> --test-level RunLocalTests --code-coverage --result-format human`

## Documentation Rules
Update documentation when any of the following changes:
- repository structure
- shared patterns
- deployment process
- architecture decision
- required local setup

## Conflict-Reduction Practices
- Keep one object or feature area per branch where possible.
- Reuse shared selectors, services, and utilities instead of duplicate implementations.
- Agree naming upfront for classes, LWCs, fields, and permission sets.
- Rebase or merge from `main` frequently for long-running work.
