# Do and Don't

## Do

- Use bind variables; never string-concatenate filters.
- Query only required fields.
- Provide bulk selector methods.
- Enforce sharing/FLS strategy explicitly.
- Add ApexDoc (`/** @description … @param … @return … */`) to every selector method.

## Don’t

- Don’t scatter inline SOQL across service/handler classes.
- Don’t return unbounded data without rationale.
- Don’t put DML or orchestration in selector classes.
