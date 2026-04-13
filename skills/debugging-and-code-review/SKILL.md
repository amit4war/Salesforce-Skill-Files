---
name: debugging-and-code-review
description: "Structured root cause analysis, debugging, and code review for Salesforce development. Use when diagnosing Apex errors, stack traces, unexpected behavior, failed processes, or test failures; reviewing existing Apex classes, triggers, or LWC for bugs and quality; investigating regressions after deployment; or integrating new features into complex systems where existing behavior must be preserved. Covers upstream/downstream analysis, log-driven diagnosis, minimal targeted fixes, test validation, and peer review sign-off."
argument-hint: "<error message or symptom> <affected component> <reproduction steps>"
---

# SKILL: Debugging and Code Review for Root Cause Analysis

## Purpose

Provide a structured, end-to-end approach for debugging code and performing code reviews with a focus on identifying root causes of issues. This skill emphasizes thorough analysis over quick patches, ensuring that fixes address underlying problems and that all upstream inputs and downstream effects are considered before changes are made.

## Use When

- **Bug Fixing**: A production or testing issue is reported (e.g., an Apex error, unexpected behavior, failed process) that requires diagnosis and resolution.
- **Code Review**: Reviewing existing code (Apex classes, triggers, Lightning Components, etc.) for potential bugs, code quality, and alignment with best practices.
- **Regression Investigation**: A previously working feature is now failing or performing poorly after a deployment or change.
- **Feature Integration**: Adding new features to a complex system where existing behavior must be preserved, requiring careful analysis of how new code interacts with current code.

## Required Inputs

- **Error Details**: Exception messages, stack traces, error IDs, or screenshots of errors.
- **Reproduction Steps**: Clear steps or tests that consistently reproduce the issue.
- **Relevant Code Artifacts**: Apex classes, triggers, Visualforce pages, Lightning components, or other code suspected to be involved.
- **Related Metadata**: Custom Settings, Custom Metadata, Validation Rules, Flows, or Process Builders that may affect code execution.
- **System Logs**: Debug logs or audit trails capturing the issue at appropriate logging levels (Apex Code, System, Database, Workflow).
- **Recent Changes**: Deployment history, merge records, commit diffs, or CI/CD logs if the issue is a regression.
- **Environment Details**: Org type (Production, Sandbox, scratch org) and any data or config differences between environments.

## Guardrails

- **No Patch-Only Fixes**: Never treat only the symptom. Identify and address the root cause; patch fixes cause recurring or masked problems.
- **Upstream Analysis**: Understand what triggers the code — user actions, scheduled jobs, integration calls, or other triggers — and verify that inputs are valid before they reach the code.
- **Downstream Analysis**: Identify all processes, systems, and integrations affected by the code's output. Confirm that a fix does not break downstream behavior.
- **Thorough Research**: Reproduce the problem and gather log evidence before changing code. No changes without evidence.
- **Minimal and Targeted Changes**: Fix only what is necessary to resolve the root cause. Avoid unrelated changes during a debugging session.
- **Maintainability**: Improve or preserve code clarity. Add inline comments explaining non-obvious root causes or fixes.
- **Preserve Existing Behavior**: Run full regression tests or all impacted user stories to validate that other features are unaffected.
- **Collaboration**: Coordinate with team members, integration owners, or stakeholders when the issue spans multiple systems.
- **Post-Deploy Monitoring**: Monitor logs and alerts after deployment to confirm resolution and detect any new issues introduced.

## Expected Outputs

- **Root Cause Analysis (RCA) Document**: Summarizes the problem, reproduction steps, relevant log excerpts, identified root cause, and the responsible code/system component.
- **Code Fix**: Updated Apex class, trigger, or component that directly addresses the root cause with clear, targeted changes.
- **Test Updates**: New or updated unit tests that fail before the fix and pass after. Tests must cover the specific edge case that caused the issue and any related scenarios discovered during debugging.
- **Impact Analysis**: Brief report on upstream/downstream behavioral changes resulting from the fix, confirming no unintended side effects.
- **Code Review Sign-Off**: Peer review record including comments raised and how they were resolved.
- **Deployment Plan**: List of metadata components to deploy, deployment order if dependencies exist, post-deployment steps (test execution, data fixes), and a rollback plan.

## Working Pattern

### Debug Agent Responsibilities

1. **Reproduce the Issue**: Replicate the bug in a controlled environment (developer sandbox or scratch org) using a specific data set or scenario. Confirm reproduction before proceeding.
2. **Gather Diagnostics**: Enable debug logging at appropriate trace levels (FINEST for Apex Code; also enable Workflow, Validation, Database as needed). Capture full logs. For LWC/Aura, use browser dev tools. For async jobs, retrieve logs from Apex Jobs.
3. **Isolate the Root Cause**: Analyze the stack trace and log sequence to pinpoint the failing location and the sequence of events. Use Developer Console Log Inspector, VS Code Apex Replay Debugger, or `System.debug` statements to step through logic.
4. **Examine Upstream Inputs**: Determine what triggered the failing code path. Check for recent changes to processes, validations, or integrations that feed into this code. Confirm whether input data meets the code's assumptions.
5. **Examine Downstream Effects**: Identify what the code is supposed to produce and whether the failure causes data inconsistency, aborted transactions, or broken subsequent processes (flows, future methods, integrations).
6. **Review the Code**: Read the suspect code and surrounding context for logic errors, improper error handling, null handling gaps, deprecated methods, bulkification issues, SOQL/DML in loops, and conformance with project standards.
7. **Consult Version History**: Use Git history to identify when the issue was introduced and the original developer intent. Diffs often reveal why code is written a certain way.
8. **Formulate Hypothesis**: Write a clear root cause statement backed by log or test evidence (e.g., "ContactsTrigger fails when `Contact.MailingState` is null because the code assumes it is always populated").
9. **Devise a Solution**: Plan the minimal fix that eliminates the root cause — null checks, corrected conditions, query adjustments, or logic reordering. Consider alternative approaches and choose the simplest viable option.
10. **Assess Fix Impact**: Before writing code, evaluate whether the change affects other callers, requires config changes, or requires data backfilling. Identify all areas needing retesting.
11. **Implement the Fix**: Apply targeted changes following project coding standards. Add comments where the root cause or fix is non-obvious.
12. **Update/Add Tests**: Write a test that fails before the fix to confirm the issue is reproducible. After fixing, ensure the test passes. Cover edge cases discovered during debugging (nulls, empty lists, bulk data, async patterns using `Test.startTest()`/`Test.stopTest()`).

### Review Agent Responsibilities

1. **Verify Root Cause Resolution**: Confirm that the identified root cause is credible and the code change directly addresses it. Challenge any fix that looks like a bandaid.
2. **Assess Upstream/Downstream Considerations**: Verify that the developer audited all callers of changed methods and all consumers of changed data structures or outputs.
3. **Code Quality and Consistency**: Check for bulkification, null safety, naming conventions, no hard-coded IDs, proper error handling, and alignment with project standards.
4. **Performance and Security**: Flag any SOQL or DML introduced inside loops. Confirm no security controls are bypassed (CRUD/FLS, sharing) and no sensitive data is exposed in logs.
5. **Run and Review Tests**: Execute tests and confirm new tests correctly validate the fix and fail without it. Review test assertions for specificity and edge-case coverage.
6. **Regression Risk**: Consider whether the change could affect other parts of the application. Request additional safeguards (feature flags, custom settings) for high-risk changes.
7. **Approve with Confidence**: Approve only when the root cause is resolved, quality criteria are met, and upstream/downstream impacts are accounted for.
8. **Mentorship**: Use the review to coach on patterns. If the bug was caused by an anti-pattern (e.g., unchecked map access), confirm the pattern is corrected throughout the codebase, not just at the single location.

## Example Scenario

**Issue**: After a recent deployment, users get "List index out of bounds: 0" when saving an Opportunity.

| Step       | Action                                                                                                                     |
| ---------- | -------------------------------------------------------------------------------------------------------------------------- |
| Reproduce  | Create an Opportunity in sandbox; capture debug log. Exception traced to `OpportunityAfterInsert` trigger, line 45.        |
| Isolate    | Handler accesses `contactList[0]` without checking if the list is empty.                                                   |
| Upstream   | Trigger fires on all Opportunity inserts. Related Contact is optional — list can legitimately be empty.                    |
| Downstream | Failure aborts entire save transaction; a downstream Account trigger also fails to complete.                               |
| Root Cause | Missing empty-list guard before index access.                                                                              |
| Fix        | Add `if (!contactList.isEmpty())` before Task creation logic.                                                              |
| Test       | New test inserts Opportunity with no contacts — fails before fix, passes after. Existing Task creation tests remain green. |
| Review     | Reviewer confirms fix logic, checks for similar patterns in other trigger handlers, approves.                              |

## Definition of Done

- [ ] Root cause is identified and fixed — not just the symptom. Developer and reviewer share a common understanding of why the issue occurred.
- [ ] Upstream and downstream impacts have been analyzed. All related components have been evaluated and no unintended side effects remain.
- [ ] Fix follows project best practices: bulk-safe, no SOQL/DML in loops, no hard-coded IDs, proper null handling, and proper error handling.
- [ ] Test coverage includes the bug scenario and related edge cases. All tests pass. The bug test fails if the fix is reverted.
- [ ] Code reviewed and approved by a peer or senior developer. All review comments are addressed.
- [ ] Documentation updated: code comments, RCA record, and any relevant design or troubleshooting docs.
- [ ] Fix deployed following change management procedures with post-deployment log monitoring complete.
- [ ] No regression or new errors observed after deployment, confirmed through logs, alerts, and user feedback.
