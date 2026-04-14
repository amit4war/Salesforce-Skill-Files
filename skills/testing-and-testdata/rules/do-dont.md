# Do and Don't

## Do

- Create all test data in test context.
- Use `@TestSetup` for shared baseline data.
- Cover positive, negative, single, and bulk scenarios.
- Use callout mocks for HTTP behavior.
- Use `System.runAs()` where security context matters.

## Don’t

- Don’t use `seeAllData=true`.
- Don’t assert only that no exception occurred.
- Don’t leave async logic untested.
- Don’t duplicate large data setup in every test method.
