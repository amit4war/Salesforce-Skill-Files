# Tech Stack and Constraints

- Salesforce Apex async runtime
- `AsyncApexJob` monitoring
- Optional callout support via Named Credentials
- Governor limits and org-level async concurrency limits

## Constraints

- Max 50 future/queueable enqueues per transaction
- Queueable chaining must terminate cleanly
- Batch concurrency is limited
- Scheduled jobs are finite and must be managed deliberately
