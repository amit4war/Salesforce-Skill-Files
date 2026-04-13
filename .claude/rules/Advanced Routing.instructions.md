---
description: Skill router — load the correct domain skill before every response. Always active for all tasks.
globs: "**/*"
---

# Skill Router — Load the Right Skill Before Every Response

Before generating any code, configuration, metadata, or guidance, identify the domain of the request and load the matching skill file. This step is **mandatory** — never skip it.

## Load Order

1. **Always load `.claude/rules/Project Guidelines Instructions.instructions.md` first** — project naming and baseline rules apply to every task.
2. Then load the domain skill below that matches the request.
3. If the request spans multiple domains, load all matching skills before proceeding.

## Skill Selection Map

| User is asking about…                                                                     | Load this skill                             |
| ----------------------------------------------------------------------------------------- | ------------------------------------------- |
| Triggers, TriggerHandler, Domain, Service, Selector scaffold                              | `skills/apex-trigger-pattern/SKILL.md`      |
| @future, Queueable, Batch Apex, Schedulable, async patterns                               | `skills/async-apex-patterns/SKILL.md`       |
| Debugging, stack traces, exceptions, root cause analysis, code review                     | `skills/debugging-and-code-review/SKILL.md` |
| LWC, Lightning Web Component, UI, SLDS, @wire, @api, Jest tests                           | `skills/lwc-data-patterns/SKILL.md`         |
| SOQL, queries, selector classes, data access, query optimization                          | `skills/soql-and-selectors/SKILL.md`        |
| Deploy, validate, sf CLI, scratch org, CI/CD, package manifest                            | `skills/deploy-and-validate/SKILL.md`       |
| Test classes, TestDataFactory, test coverage, assert strategies                           | `skills/testing-and-testdata/SKILL.md`      |
| Naming, Flow, OmniStudio, bypass mechanism, project standards, metadata, any new artifact | `skills/configuration-patterns/SKILL.md`    |

## Rules

- Do not generate any artifact before loading the applicable skill(s).
- If no domain skill clearly matches, still load Project Guidelines as the baseline guardrail.
- When a skill is loaded, follow its guardrails, naming conventions, and definition of done without exception.
- All generated code and metadata must satisfy the Definition of Done in the Project Guidelines.
