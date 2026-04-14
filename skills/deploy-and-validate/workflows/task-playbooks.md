# Task Playbooks

## Playbook A: Validate a Feature Branch

1. Generate scoped manifest.
2. Run validate with selected test level.
3. Capture test outputs for PR.

## Playbook B: Deploy to Sandbox/UAT

1. Revalidate latest branch state.
2. Deploy same validated scope.
3. Run post-deploy smoke tests.

## Playbook C: Triaging Deployment Failure

1. Collect deploy job ID and failing components.
2. Classify failure type (metadata, tests, permissions).
3. Fix root cause and revalidate.

## Playbook D: Destructive Change Deployment

1. Deploy additive dependencies first.
2. Validate business impact.
3. Run destructive package as separate step.
4. Execute rollback contingency if issues appear.
