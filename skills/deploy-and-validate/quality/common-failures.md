# Common Failures

1. Wrong target org alias
   - Prevention: require alias confirmation before command execution.

2. Over-scoped manifest
   - Prevention: include only changed components and dependencies.

3. Incorrect test level
   - Prevention: use risk-based matrix and avoid undershooting.

4. Destructive changes without dependency checks
   - Prevention: separate destructive deploy and include rollback.

5. Re-running full deploy after partial success
   - Prevention: inspect failed components and resume targeted execution.
