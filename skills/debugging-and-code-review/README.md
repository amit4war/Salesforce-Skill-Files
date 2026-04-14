# Debugging and Code Review Skill

Production-ready guidance for root cause analysis, targeted bug fixes, and structured code review.

## When To Use

- Diagnosing Salesforce errors or failing behavior
- Investigating regressions after deployment
- Reviewing risky Apex, LWC, trigger, or metadata changes
- Validating that a fix resolves cause, not just symptom

## When Not To Use

- User cannot provide any symptom, evidence, or reproducible path
- Request is only to rubber-stamp code without analysis

## Quick Start

1. Reproduce the issue.
2. Gather logs and relevant code.
3. Form a root cause hypothesis.
4. Implement the smallest viable fix.
5. Add a failing-then-passing test.
6. Review upstream and downstream impact.
