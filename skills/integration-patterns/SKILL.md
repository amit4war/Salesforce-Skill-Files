---
name: integration-patterns
description: "Design and implement Salesforce integration patterns: REST/SOAP callouts, Platform Events, Change Data Capture (CDC), Named Credentials, and error/retry contracts. Use when building outbound callouts to external APIs, consuming external webhooks, publishing or subscribing to Platform Events, implementing CDC-based real-time sync, or designing retry and error handling for asynchronous integration flows."
argument-hint: "<integration type: callout|platform-event|cdc|inbound-rest> <external system> <authentication method>"
---

# SKILL: Integration Patterns

## Purpose

Design and implement secure, resilient Salesforce integration patterns that use Named Credentials
for authentication, async patterns for non-blocking execution, and explicit error/retry contracts
to prevent silent failures.

## Use When

- Calling an external REST or SOAP API from Apex.
- Publishing Platform Events to notify external or internal consumers.
- Subscribing to Platform Events or Change Data Capture (CDC) events.
- Receiving inbound API calls (REST resource, site, Connected App).
- Designing retry behavior, error logging, or compensating transactions for async integrations.
- Replacing outbound message session ID usage with OAuth-based authentication (Spring '26 change).

## When Not To Use

- Requirement is purely internal Apex-to-Apex communication with no external system.
- Configuration-only change (Named Credential setup) with no code — use configuration-patterns instead.
- OmniStudio Integration Procedure is the correct container — use omni-studio-patterns.

---

## Required Inputs

- Integration type (REST callout | SOAP callout | Platform Event pub | Platform Event sub | CDC | Inbound REST)
- External system name and endpoint pattern (or Named Credential alias)
- Authentication method (OAuth 2.0 JWT | OAuth 2.0 Client Credentials | Basic — via Named Credential only)
- Invocation context (trigger, queueable, batch, LWC imperative)
- Error handling and retry requirements
- Data volume and latency expectations

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent endpoint URLs, credential names, or API field names.

---

## Authentication Rules

- **Never** use session IDs for outbound integration — Salesforce removed this capability (week of Feb 23, 2026).
- **Always** use Named Credentials for endpoint URLs and authentication. Never hardcode base URLs.
- OAuth 2.0 is the required authentication standard for all new integrations.
- Secrets and tokens must never appear in Apex code, logs, or Custom Metadata.

---

## Callout Pattern Rules

- Callouts are not allowed after DML in the same transaction — use `@future(callout=true)` or Queueable.
- Always use `https://` endpoints. `http://` is a SonarQube Critical blocker.
- All Queueable callout classes must implement a Finalizer (`System.attachFinalizer()`).
- Use `HttpCalloutMock` in all test classes that cover callout code.
- Set explicit timeout values on `HttpRequest.setTimeout()` — never rely on platform default.

```apex
public class ExternalApiQueueable implements Queueable, Database.AllowCallouts {
    private List<Id> recordIds;

    public ExternalApiQueueable(List<Id> recordIds) {
        this.recordIds = recordIds;
    }

    public void execute(QueueableContext ctx) {
        System.attachFinalizer(new ExternalApiQueueableFinalizer());
        HttpRequest req = new HttpRequest();
        req.setEndpoint('callout:MyNamedCredential/api/v1/resource'); // Named Credential only
        req.setMethod('POST');
        req.setTimeout(10000); // explicit timeout in ms
        req.setHeader('Content-Type', 'application/json');
        // req.setBody(JSON.serialize(payload));
        Http http = new Http();
        HttpResponse res = http.send(req);
        if (res.getStatusCode() != 200) {
            // log failure with status code and body — do not throw silently
        }
    }
}
```

---

## Platform Event Pattern Rules

- Use Platform Events for fire-and-forget, decoupled communication between systems or orgs.
- Publish with `EventBus.publish()` and handle failures via `Database.SaveResult`.
- Subscribe via Apex trigger on the Platform Event object (separate trigger, follows trigger-handler pattern).
- Use `ReplayId` for durable subscription and replay of missed events.

```apex
// Publisher
MyEvent__e evt = new MyEvent__e(Payload__c = JSON.serialize(data));
Database.SaveResult sr = EventBus.publish(evt);
if (!sr.isSuccess()) {
    // log sr.getErrors()
}
```

---

## Error Handling Contract

Every integration must define:

| Scenario                    | Required Behavior                                                      |
| --------------------------- | ---------------------------------------------------------------------- |
| HTTP 4xx (client error)     | Log request + response body; do not retry                              |
| HTTP 5xx (server error)     | Log and retry up to 3 times with exponential backoff via Queueable     |
| Timeout / network failure   | Log exception; enqueue retry job                                       |
| Partial success (batch POST)| Log failed items individually; process successful items normally       |
| Authentication failure (401)| Log and alert; do not retry — credential or config issue               |

---

## Guardrails

- Never store integration responses in `Database.Stateful` as non-serializable types.
- Keep callout logic in a dedicated Queueable or `@future` — never directly in a trigger body.
- Mock all callouts in tests using `HttpCalloutMock` — never call real endpoints in test classes.
- Use Named Credentials for all authentication — never embed credentials in code or metadata.
- Log HTTP status codes, response bodies, and error messages for every failure path.
- Add ApexDoc to every integration class and method.

---

## Start Here

1. Read `README.md` for scope and quick start.
2. Load context from `context/`.
3. Apply guardrails from `rules/`.
4. Follow `workflows/execution-steps.md`.
5. Validate with `quality/validation-checklist.md`.

## Mandatory Output Contract

Every integration task output must include:

- Integration type and authentication approach
- Named Credential alias used
- Error handling and retry contract
- Test coverage with `HttpCalloutMock` or Platform Event test patterns
- Deployment and rollback notes

**Response sections, in this order, using markdown `##` headers:**
1. **Scope Confirmation** — integration type, external system, auth method, invocation context
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — callout class, Named Credential, error handler, retry logic
4. **Test Evidence** — mock strategy, scenarios covered (success/4xx/5xx/timeout), coverage estimate
5. **Deployment and Rollback** — Named Credential setup steps, deploy order, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.
