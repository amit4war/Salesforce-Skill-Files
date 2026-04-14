---
name: lwc-data-patterns
description: "Build Lightning Web Components that retrieve, render, and update Salesforce data using SLDS 2.x, modern UX patterns, and reusable shared components. Use when creating a new LWC, refactoring for better data handling or UX alignment, standardizing UI elements (headers, forms, picklists, modals, toasts) across the project, adopting @wire adapters, Lightning Data Service, or imperative Apex, implementing loading/empty/success/error states, or aligning multi-developer UI work. Covers state management, SLDS layout patterns, Lightning base components, LMS, NavigationMixin, Jest testing, and Apex security."
argument-hint: "<component purpose> <data source: wire|lds|apex> <target surface: app-page|record-page|modal|flow>"
---

# SKILL: LWC Data and UX Patterns

## Purpose

Build Lightning Web Components that retrieve, render, and update Salesforce data using the latest Salesforce LWC design standards, including the most recent SLDS (Salesforce Lightning Design System) updates, modern UX patterns, and reusable shared components. This ensures consistency, accessibility, and maintainability across the application.

## Use When

- Creating a new LWC.
- Refactoring an existing component for better data handling or UX alignment.
- Aligning UI work across multiple developers for a consistent look and feel.
- Enforcing a unified UX structure across related components.
- Standardizing common UI elements (headers, forms, picklists, multi-selects, modals, toasts) across the entire project.
- Adopting new Salesforce LWC design patterns or SLDS updates (e.g., SLDS 2.x).

## Required Inputs

- Component purpose and target user flow.
- Data source choice: wire adapter, Lightning Data Service (LDS), or Apex.
- Required UI states: loading, empty, success, error.
- Target surface: app page, record page, utility bar, modal, or flow screen.
- Required user actions: view-only, edit, save/cancel, refresh, delete.
- Inventory of existing shared LWCs in the project (`c/` namespace) to reuse before creating a new component.
- Latest SLDS version guidelines and available LWC base components (e.g., `lightning-record-form`, `lightning-modal`, `lightning-*`).

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent values, placeholder names, or sample field names.
> See `examples/example-prompts.md` and `examples/example-outputs.md` for expected delivery shape.

- Apex `*Test.cls` updates for any changed or new Apex methods used by the LWC.

---

## Start Here

1. Read `README.md` for scope and quick-start.
2. Load context from:
   - `context/project-context.md`
   - `context/tech-stack.md`
3. Apply constraints from:
   - `rules/do-dont.md`
   - `rules/authoring-rules.md`
4. Execute `workflows/execution-steps.md`.
5. Apply scenario playbook from `workflows/task-playbooks.md`.
6. Validate against `quality/validation-checklist.md`.

## Mandatory Output Contract

Every LWC task output must include:

- Data source strategy (`wire`, LDS, or Apex) with rationale
- UI state handling plan (loading/empty/success/error)
- Reuse decision (existing component vs new)
- Security notes for any Apex-backed data path
- Jest and Apex test coverage notes

**Response sections, in this order, using markdown `##` headers:**

1. **Scope Confirmation** — component purpose, target surface, data source chosen
2. **Files Changed** — full relative path, type of change (New | Modified | Deleted)
3. **Implementation Notes by Layer** — template, controller, Apex method decisions
4. **Test Evidence** — Jest scenarios covered, Apex test coverage for any new methods
5. **Deployment and Rollback** — deploy order, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.
