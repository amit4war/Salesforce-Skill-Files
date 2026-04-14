# Tech Stack and Constraints

- Apex selector classes
- SOQL query language
- Salesforce sharing, CRUD/FLS controls

## Constraints

- Avoid SOQL duplication across layers.
- Keep selectors query-only (no DML/business logic).
- Use bulk signatures (`Set<Id>`, filter objects) where relevant.
