---
name: omni-studio-patterns
description: "Design and implement OmniStudio components: OmniScripts, DataRaptors (Extract/Load/Transform/Turbo), Integration Procedures, and FlexCards. Use when building step-by-step guided user flows, ETL processes, server-side API orchestration, or dynamic 360-degree UI dashboards in Salesforce OmniStudio. Follows naming conventions, size limits, and best practices from Project Guidelines §10."
argument-hint: "<component type: omniscript|dataraptor|ip|flexcard> <functional area> <data source or target>"
---

# SKILL: OmniStudio Patterns

## Purpose

Design and implement reusable, maintainable OmniStudio components that stay within platform
size limits, follow canonical naming conventions, and keep business logic in the correct layer.

## Use When

- Building a step-by-step guided user flow (OmniScript).
- Extracting, transforming, or loading data between Salesforce and external systems (DataRaptor).
- Orchestrating server-side API calls and complex backend logic (Integration Procedure).
- Building dynamic, data-driven UI dashboards or record-page views (FlexCard).
- Extending or refactoring an existing OmniStudio component.
- Applying Project Guidelines §10 naming or structural standards to new or existing components.

## When Not To Use

- Requirement is a pure Apex trigger or Flow automation with no UI or orchestration layer.
- Integration logic is simple enough for a single synchronous Apex callout.
- The component has no reuse potential and would be easier as a standard LWC.

---

## Required Inputs

- Component type (OmniScript | DataRaptor | Integration Procedure | FlexCard)
- Functional area and business goal
- Data source(s) and target object(s)
- User context (which profile/permission set will use this component)
- Expected data volume (for DataRaptor size-limit compliance)
- Whether external API calls are involved (determines IP vs DR)

> **Missing Input Protocol:** If any Required Input above is absent, halt and ask the caller to supply it before generating any artifact. Do not invent component names, API names, or field mappings.

---

## Naming Conventions

| Component Type   | Convention                      | Example                               |
| ---------------- | ------------------------------- | ------------------------------------- |
| OmniScript       | PascalCase, prefixed by area    | `OrderReservationCreation`            |
| DR Extract       | `DR_Extract_<Feature>`          | `DR_Extract_OrderDetails`             |
| DR Load          | `DR_Load_<Feature>`             | `DR_Load_SiteInfo`                    |
| DR Transform     | `DR_Transform_<Feature>`        | `DR_Transform_OrderDetails`           |
| DR Turbo Extract | `DR_TurboExtract_<Feature>`     | `DR_TurboExtract_DailyCapacitiesData` |
| IP Extract       | `IP_Extract_<Feature>`          | `IP_Extract_OrderDetails`             |
| IP Load          | `IP_Load_<Feature>`             | `IP_Load_SiteInfo`                    |
| IP Orchestrate   | `IP_Orchestrate_<Feature>`      | `IP_Orchestrate_DailyCapacitiesData`  |
| FlexCard         | `FC_<ObjectOrArea>_<Feature>`   | `FC_Site_Creation`                    |

---

## Size Limits (Hard Constraints)

| Component      | Limit                                              |
| -------------- | -------------------------------------------------- |
| DR Extract     | ≤5 queries, minimal formulas, ≤30 output fields    |
| DR Transform   | ≤200 output fields                                 |
| DR Load        | ≤3 objects                                         |
| IP Action Sets | Minimize; group related API calls in one Action Set |

---

## Component Role Boundaries

| Component              | DO                                                  | DO NOT                                               |
| ---------------------- | --------------------------------------------------- | ---------------------------------------------------- |
| OmniScript             | Step-by-step user interaction, subflow delegation   | Heavy business logic; direct API calls               |
| DataRaptor             | ETL — extract, transform, load                      | Orchestration; complex conditional logic              |
| Integration Procedure  | Server-side orchestration, API calls, large datasets| Heavy transformation (use DR for that)               |
| FlexCard               | Display-only dynamic UI                             | Business logic; use IP/OmniScript for that           |

---

## Guardrails

- Never hardcode IDs, record type names, or configurable values — use Custom Metadata or Custom Settings.
- Separate extraction and transformation into distinct DataRaptors.
- Default null values in DataRaptors to prevent transformation errors.
- Isolate DML in the main flow/OmniScript; pass assignments back from subflows.
- Use Error Step (OmniScript) and Fault Paths (IP) for error handling.
- For screen flows requiring system context: isolate only those pieces into system-context subflows.
- Use DataRaptor Turbo Extract only for high-volume extraction with chunked retrieval.
- Add ApexDoc or equivalent description comments to all Integration Procedure action sets.

---

## Start Here

1. Read `README.md` for scope and quick start.
2. Load context from `context/`.
3. Apply guardrails from `rules/`.
4. Follow `workflows/execution-steps.md`.
5. Validate with `quality/validation-checklist.md`.

## Mandatory Output Contract

Every OmniStudio task output must include:

- Component type and naming rationale
- Size-limit compliance check
- Layer responsibility clarification (which logic lives where)
- Test or validation steps
- Deployment notes

**Response sections, in this order, using markdown `##` headers:**
1. **Scope Confirmation** — component type, functional area, data flow
2. **Files / Components Changed** — component API names and change type (New | Modified)
3. **Implementation Notes by Layer** — per-component design decisions
4. **Test Evidence** — validation steps or test scenarios
5. **Deployment and Rollback** — deploy order, rollback steps

**Null rule:** If a section has nothing to report, output `N/A — [one-line reason]`. Never omit a section.
