# Common Failures and Prevention

## 1) Trigger Contains Business Logic

Failure:

- Logic is coded directly inside trigger body.

Prevention:

- Keep trigger limited to context build, toggle check, and handler routing.

## 2) SOQL or DML in Loops

Failure:

- Query/DML executed per record, causing limit failures.

Prevention:

- Use `Set`/`Map` preprocessing and collection-level operations.

## 3) Missing Field-Change Guards

Failure:

- Expensive logic runs on every update.

Prevention:

- Compare `oldMap` and `newMap` and exit early when monitored fields did not change.

## 4) Recursion and Re-Entry Loops

Failure:

- Trigger updates related/same-object records and re-enters repeatedly.

Prevention:

- Add recursion guard keys tied to operation and record scope.

## 5) Incomplete Security Handling

Failure:

- Data access ignores sharing/CRUD/FLS constraints.

Prevention:

- Declare sharing intent and enforce object/field access strategy explicitly.

## 6) Weak Deployment Safety

Failure:

- No kill switch, no rollback, no post-deploy checks.

Prevention:

- Implement toggle control and publish rollback steps before deployment.

## 7) Insufficient Tests

Failure:

- Tests only cover happy path.

Prevention:

- Require bulk, field-change, recursion, and toggle-off tests for acceptance.
