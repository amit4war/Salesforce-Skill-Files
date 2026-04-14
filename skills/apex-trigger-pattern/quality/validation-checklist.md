# Validation Checklist

Run this checklist before finalizing any trigger task.

## 1) Scope and Correctness

- [ ] Object and event scope matches request.
- [ ] All stated business rules are implemented.
- [ ] Non-functional requirements are addressed (performance/security/auditability).

## 2) Architecture and Modularity

- [ ] Trigger is thin and routes only.
- [ ] Handler, domain, service, and selector responsibilities are clear.
- [ ] No cross-layer responsibility leakage.

## 3) Bulk and Limits Safety

- [ ] No SOQL in loops.
- [ ] No DML in loops.
- [ ] Logic works for 1 record and 200 records.
- [ ] Field-change gating is used where appropriate.

## 4) Security and Permissions

- [ ] Sharing model decisions are explicit.
- [ ] CRUD/FLS implications are considered.
- [ ] Sensitive data is not logged or leaked.

## 5) Operational Safety

- [ ] Trigger toggle strategy exists and is test-covered.
- [ ] Recursion guard exists where needed.
- [ ] Deployment steps are documented.
- [ ] Rollback steps are documented and feasible.

## 6) Test Quality

- [ ] Happy path covered.
- [ ] Bulk path covered.
- [ ] Field-change path covered.
- [ ] Recursion path covered.
- [ ] Toggle disabled path covered.

## 7) Final Output Quality

- [ ] Response is explicit, actionable, and structured.
- [ ] File list and key decisions are included.
- [ ] Risks and next checks are clearly stated.
