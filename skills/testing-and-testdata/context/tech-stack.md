# Tech Stack and Constraints

- Apex test framework
- `@TestSetup`, `Test.startTest()`, `Test.stopTest()`
- `HttpCalloutMock` and static resource mocks
- `System.runAs` for user-context tests

## Constraints

- No `seeAllData=true`
- Deterministic test data only
- Meaningful assertions required
