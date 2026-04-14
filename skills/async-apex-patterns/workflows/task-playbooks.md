# Task Playbooks

## Future Method

- Use only for simple fire-and-forget operations.
- Restrict parameters to primitives or primitive collections.
- Avoid when monitoring, chaining, or complex state is required.

## Queueable

- Use for most new async patterns.
- Pass structured state safely.
- Chain only when one next step is required.

## Batch Apex

- Use for large datasets and chunked processing.
- Keep `execute()` bulk-safe and repeatable.
- Use `finish()` for summary or next-step orchestration.

## Schedulable

- Use to launch queueable or batch on a schedule.
- Keep scheduled classes thin.
- Document CRON cadence and operational ownership.
