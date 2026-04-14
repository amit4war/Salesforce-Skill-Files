# Execution Steps

1. Confirm branch, target org alias, and deployment scope.
2. Classify risk (low/medium/high/critical).
3. Choose test level based on risk.
4. Generate or verify manifest/scope.
5. Run validation command first.
6. Analyze failures and fix root causes.
7. Run deploy command after successful validation.
8. Capture outputs: job ID, test results, status.
9. Execute post-deploy smoke checks.
10. Publish rollback plan and evidence.

## Expected Output Per Run

- Validation command + result
- Deploy command + result
- Test summary
- Risk note
- Rollback note
