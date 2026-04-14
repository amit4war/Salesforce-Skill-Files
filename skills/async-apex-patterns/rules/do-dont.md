# Do and Don't Guardrails

## Do

- Do choose Batch for large-volume processing.
- Do prefer Queueable over future for new complex async work.
- Do use Named Credentials for callouts.
- Do guard against duplicate execution.
- Do log or surface failures intentionally.
- Do attach a Finalizer (`System.attachFinalizer()`) to every Queueable class.
- Do add ApexDoc (`/** @description … @param … @return … */`) to every `public` and `global` class, method, and property.

## Don't

- Don't enqueue one job per record.
- Don't place SOQL or DML in loops.
- Don't chain indefinitely.
- Don't assume execution order without explicit control.
- Don't leave retry behavior undefined.
