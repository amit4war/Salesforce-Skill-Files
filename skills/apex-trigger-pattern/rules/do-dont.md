# Do and Don't Guardrails

## Do

- Do keep one trigger per object.
- Do route by `Trigger.operationType` or equivalent clear context routing.
- Do process collections and use `Set`/`Map` preprocessing.
- Do gate expensive logic with field-change checks (`oldMap` vs `newMap`).
- Do isolate queries in selectors and orchestration in services.
- Do add recursion guards when follow-up DML can re-enter logic.
- Do include trigger bypass toggle checks for controlled deployments.
- Do test both enabled and disabled automation states.
- Do add ApexDoc (`/** @description … @param … @return … */`) to every `public` and `global` class, method, and property.

## Don’t

- Don’t put SOQL or DML inside loops.
- Don’t implement business logic directly in trigger bodies.
- Don’t modify `Trigger.new` records in `after` contexts.
- Don’t assume single-record execution.
- Don’t skip CRUD/FLS and sharing considerations.
- Don’t perform risky cross-object updates without dependency analysis.
- Don’t deploy without rollback instructions.

## Security and Unsafe Actions

- Never hardcode credentials, tokens, or secrets.
- Never expose sensitive data in debug logs or thrown errors.
- Avoid destructive metadata/data operations unless user explicitly requests and confirms scope.
- For risky actions (mass updates/deletes), require dry-run impact summary first.
