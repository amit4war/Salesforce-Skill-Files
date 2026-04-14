# Tech Stack and Constraints

## Tooling

- Salesforce CLI (`sf`)
- SFDX project configuration (`sfdx-project.json`)
- Manifest-driven and source-dir deploy patterns

## Constraints

- Use explicit `--test-level`.
- Validate medium/high risk before deploy.
- Keep API version aligned with project config.
- Handle destructive changes in separate controlled steps.
