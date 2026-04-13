---
name: soql-and-selectors
description: "Develop reusable, secure, and governor-limit-aware SOQL query logic using the Selector pattern in Apex. Use when writing new data access queries in service or trigger logic, refactoring duplicate inline SOQL into a selector class, enforcing CRUD/FLS security centrally, optimizing query performance with selective filters and limits, or adopting Apex Enterprise Patterns that separate query logic from business logic. Covers bind variables, minimal field selection, bulk Set<Id> patterns, sharing context, WITH SECURITY_ENFORCED, stripInaccessible, naming conventions, and selector unit testing."
argument-hint: "<SObject API name> <filter conditions> <security context: user|system>"
---

# SKILL: SOQL and Selector Design

## Purpose

Develop reusable, secure, and governor-limit-aware query logic using the Selector pattern in Apex. This ensures that all data retrieval is centralized, consistent, and optimized for performance and maintainability across service and domain layers. By encapsulating SOQL in selector classes, you achieve:

- **Better Reusability**: Queries can be called from multiple places without duplication.
- **Easier Maintenance**: Field sets and conditions are defined in one place, reducing drift.
- **Governance**: All queries are optimized (selective fields, proper filters, limits) and easier to review for security (FLS/CRUD) compliance.
- **Testability**: Selector methods can be unit tested independently and mocked in tests of higher-level logic.

## Use When

- **Adding New Queries**: Whenever writing Apex that needs to fetch data (especially in service or trigger logic), create or use a selector method instead of inline SOQL.
- **Refactoring Duplicate Queries**: If the same or similar SOQL appears in multiple places, consolidate it into a selector class method.
- **Scaling Data Access**: In enterprise applications following Apex Enterprise Patterns, to separate query logic from business logic (domain/services) for clarity and consistency.
- **Optimizing Performance**: To review and improve data access patterns — ensuring filters are selective, proper indexing is used, and unnecessary fields are avoided.
- **Enforcing Security**: To centralize CRUD/FLS checks around queries so that all data access abides by security requirements (e.g., using `with sharing` or `stripInaccessible` in selectors as needed).

## Required Inputs

- **Object and Fields**: The specific SObject and the minimal set of fields required for the operation.
- **Filter Conditions**: The criteria to retrieve relevant records (e.g., `Status = 'Open' AND CloseDate < THIS_YEAR`).
- **Sort Order**: Whether results need ordering (e.g., `ORDER BY CreatedDate DESC`) for consistent processing or UI presentation.
- **Cardinality and Usage**: How many records are expected (single record, small list, potentially large list) and how the result will be used (display, batch process, existence check).
- **Security Context**: Whether the query runs in user or system mode. Whether to enforce sharing (via `with sharing`) or manually handle FLS.
- **Data Relationships**: If related data is needed (child or parent records), determine whether to use subqueries or separate selectors.

## Guardrails

- **Use Bind Variables**: Always use Apex variables in the `WHERE` clause instead of string concatenation. This prevents SOQL injection and ensures proper quoting.
- **Select Minimal Fields**: Query only the fields necessary for the logic. Extra fields increase heap usage and risk FLS issues. Update the selector when additional fields are needed — do not query again elsewhere.
- **Apply Limits**: Use a `LIMIT` clause when expecting one record or a small subset. Document when a query is intentionally unbounded (e.g., an export batch) and ensure it is acceptable.
- **Reuse Selectors**: Do not write inline SOQL in multiple places. Add a method to the appropriate selector class (e.g., `AccountSelector`) and have all consumers call it.
- **Selectors Are Query-Only**: Selector classes should contain only retrieval logic (SOQL statements), returning SObjects or collections. No DML or business logic belongs in selectors; keep that in service or domain classes.
- **Explicit Security Handling**:
  - Use `with sharing` in selector classes if queries should respect sharing rules (most selectors should, unless an explicit system-context decision is documented).
  - Consider `Security.stripInaccessible` for field-level read enforcement when returning records to a UI context.
  - If a selector intentionally queries records the user may not have access to, document and justify the decision.
- **Bulk and Performance**: Design selector methods to accept collections of inputs (e.g., `Set<Id>`) and use a single `IN` clause query rather than one query per Id. Ensure no selector calls another selector inside a loop.
- **Naming Conventions**: Name selector methods clearly by intent:
  - `findByIds(Set<Id> ids)` — general bulk fetch by Id.
  - `byAccountId(Id accountId)` — filtering by a foreign key.
  - `openCasesByAccountIds(Set<Id> accountIds)` — child records meeting specific criteria.
- **No Orphaned Filters**: All filter logic lives in the selector. Upstream code must not add extra filtering on returned results — pass parameters to the selector's `WHERE` clause instead.
- **Document Row Expectations**: Make clear in comments or method names if a method returns at most one record (and how ties are handled) or could return multiple.
- **Governor Limits**: Each selector method performs exactly one SOQL query. If a consumer might need to call it in a loop, provide a bulk method accepting a `Set` of parent IDs.
- **Query Planning**: Use selective filters (indexed fields) whenever possible. If a filter on a high-volume object is not selective, consider formula fields for selectivity, or use `LIMIT` with client-side filtering only when the data set is known to be small.

## Expected Outputs

- **Selector Class Changes**: New methods in an existing selector class or a new selector class if none exists for that object. Example:
  ```apex
  public with sharing class AccountSelector {
      public static List<Account> byOwnerIds(Set<Id> ownerIds) {
          if (ownerIds == null || ownerIds.isEmpty()) {
              return new List<Account>();
          }
          return [
              SELECT Id, Name, Industry
              FROM Account
              WHERE OwnerId IN :ownerIds
          ];
      }
  }
  ```
- **Updated Call Sites**: Apex classes that originally had inline queries now call the selector method. Example — before:
  ```apex
  List<Account> accts = [SELECT Id, Name FROM Account WHERE OwnerId = :ownerId];
  ```
  After:
  ```apex
  List<Account> accts = AccountSelector.byOwnerIds(new Set<Id>{ ownerId });
  ```
- **Unit Tests for Selectors**: Tests covering:
  - Basic functionality: method returns the expected records and no others.
  - Empty inputs: passing an empty or null set returns an empty list without executing a query.
  - Security: if the selector enforces sharing, test with a user who should not see certain records and verify they are excluded.
  - Edge cases: boundary conditions for date filters, missing related records, etc.
- **Documentation/Comments**: A brief comment on each method where usage is non-obvious or where special considerations exist (e.g., "returns at most 1 case per account, prioritizing newest by CreatedDate").
- **Consistent Pattern Adoption**: Service and domain classes call selectors for data retrieval. Trigger handlers, batch classes, and controllers updated to use selectors rather than inline SOQL.
- **No Functional Regression**: Application behavior is unchanged (except performance or security improvements). All existing tests still pass.

## Working Pattern

1. **Identify the Need**: When new data must be fetched, clarify how that data will be used and whether similar queries already exist in a selector.
2. **Determine Reusability**: If the query logic might be needed elsewhere — even likely — it belongs in a selector method. For consistency, use selectors even for one-off queries.
3. **Design the Selector Method**: Write the method signature to clearly reflect intent (e.g., `openOppsByAccountIds(Set<Id> acctIds)`). Decide on return type: `List<SObject>` for general retrieval, `Map<Id, SObject>` if callers need keyed access, or a single SObject for unique lookups.
4. **Write the SOQL**: Implement guard clauses (return empty immediately for null/empty inputs). Use descriptive variable names and clear formatting. Example:
   ```apex
   public static List<Opportunity> openOppsByAccountIds(Set<Id> accountIds) {
       if (accountIds == null || accountIds.isEmpty()) return new List<Opportunity>();
       return [
           SELECT Id, Name, StageName, CloseDate, AccountId
           FROM Opportunity
           WHERE AccountId IN :accountIds
             AND IsClosed = FALSE
           ORDER BY CloseDate ASC
       ];
   }
   ```
5. **Bulk Consideration**: Ensure the method handles large input sets efficiently. If expecting thousands of records in the result, assess whether pagination or additional filters are needed.
6. **Integrate into Business Logic**: Replace inline SOQL in service/domain code with calls to this selector. Adjust the calling code for any return type differences.
7. **Security Measures**: Confirm whether the class should be `with sharing`. If needing field/object CRUD enforcement, use `WITH SECURITY_ENFORCED` in the SOQL (Spring '20+) or `Security.stripInaccessible` on results. Document the chosen approach.
8. **Test the Selector**:
   - Create test data the query should retrieve.
   - Assert the selector returns exactly the expected records.
   - Test edge conditions: null IDs, IDs with no matching records, boundary dates.
   - If using `WITH SECURITY_ENFORCED`, test with a user lacking field access and verify the expected `System.QueryException` is thrown (or use `stripInaccessible` to avoid the exception).
9. **Refactor Iteratively**: As other features surface similar queries, continue to move them into the appropriate selector methods. Selectors become the single source of truth for all SOQL.

## Definition of Done

- [ ] **Correctness**: The selector returns expected data for all scenarios. It handles empty/null inputs without throwing unhandled exceptions.
- [ ] **No Duplicate SOQL**: No instances of the same query logic are scattered across different classes. All such logic routes through the selector. Overlapping queries across selectors are rationalized.
- [ ] **Security Compliance**: Data access respects org security settings. User-facing queries run in sharing context or use explicit security enforcement. Any sharing bypass is documented and justified.
- [ ] **Performance**: The query uses indexed filters where possible and does not pull unnecessary data. No SOQL inside loops at call sites after refactoring to selectors.
- [ ] **Reusability**: Another developer can find and use the selector method. Each selector class is organized logically (one selector per object or theme, e.g., `CaseSelector` for Case queries).
- [ ] **Maintainability**: Changing a query (adding a field, adjusting a filter) requires altering one place — the selector — and its tests. No scattered copies exist to drift.
- [ ] **Documentation**: Method intent and any special considerations are clear from the name and optional comment. The selector class may have a header comment describing its role.
- [ ] **Test Coverage**: The selector class is covered by unit tests (targeting 90–100% of selector code). Higher-level service/domain tests still pass, confirming integration is intact.
- [ ] **Integration**: All application layers use selectors for data access. The team understands and enforces this pattern in code reviews — new SOQL goes into selectors, not scattered inline.
