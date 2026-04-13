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

## Guardrails

- Use SLDS 2.x design patterns and design tokens (CSS custom properties) for styling; avoid custom CSS that conflicts with SLDS. This ensures consistency and easy theming (e.g., for dark mode or branding).
- Prefer `@wire` adapters (like `getRecord`, `getListUi`) for read-only data flows when possible, and Lightning Data Service for standard record CRUD (`lightning-record-form` or `lightning-record-edit-form`) to minimize Apex calls.
- Use imperative Apex calls for complex create/update operations or when server-side business logic is needed, but keep the UI responsive with spinners and optimistic UI where appropriate.
- Implement a consistent state model in every component: clearly handle loading (spinner or skeleton), empty (no records or data), success (data loaded), and error (message with error details).
- Use a consistent layout structure in the template: order sections as header, content, actions, then feedback (messages/toasts).
- Place actions (Save, Cancel, Edit, Delete, Refresh) in consistent locations (form buttons in footer, refresh in header). Use standard labels ("Save", "Cancel") for uniformity.
- Reuse shared components before creating custom inputs or picklists. Check for `c/commonInput`, `c/commonPicklist`, or equivalent shared LWCs first.
- Use Lightning base components (`lightning-*`) for form controls, buttons, layout grids, and datatables. They follow SLDS out of the box (e.g., `lightning-combobox` for dropdowns, `lightning-dual-listbox` for multi-select, `lightning-datatable` for tables).
- **Modals and Dialogs**: Use `lightning-modal` (with `<lightning-modal-header>`, `<lightning-modal-body>`, `<lightning-modal-footer>`) instead of custom modal markup. Use `LightningAlert`, `LightningConfirm`, and `LightningPrompt` from `lightning/alert`, `lightning/confirm`, and `lightning/prompt` for alert/confirmation dialogs instead of native browser dialogs. Use `ShowToastEvent` for user notifications.
- Ensure accessibility: use semantic HTML, provide labels for all form inputs, ensure keyboard navigation and focus management in modals, and use ARIA attributes for dynamic content and custom elements.
- Keep business logic out of the component when possible. Use Apex for complex data transformations or multi-step operations. Use utility modules for shared calculations.
- Avoid duplicating logic across components. If multiple LWCs need similar behavior, create a shared helper module or service.
- Keep JavaScript methods concise and focused. Ensure all branches of logic (especially error handling) are accounted for.
- Use Lightning Message Service (LMS with `MessageContext`, `publish`, `subscribe`) for cross-component communication instead of custom pub-sub implementations.
- Use `NavigationMixin.Navigate` for in-app navigation (record page, object page redirections) instead of building custom logic.
- Add Jest tests for any component with conditional logic or complex behavior.
- For all Apex methods called by LWCs, ensure corresponding Apex test methods cover expected outcomes and error cases. Apex classes must enforce CRUD/FLS security and be bulkified when processing lists of records.

## Expected Outputs

- Updated LWC files:
  - `.html` template with sections for header, body, footer, and conditional markers for UI states (spinner for loading, error panel for errors, empty-state message).
  - `.js` controller using `@wire` adapters or imperative Apex calls, with `@api`/`@track` properties for data and state, and graceful promise/error handling (`try/catch` or `.catch()`).
  - `.js` or separate utility modules for shared logic (data formatting, validation).
  - `.js-meta.xml` configured with correct targets, exposure, object permissions, and the latest API version.
  - `.css` using SLDS utility classes or design tokens for custom styling only where necessary, scoped to avoid overriding SLDS globally.
- Explicit empty/error/loading states: `<lightning-spinner>` or skeleton placeholders while loading; placeholder message or illustration when empty; styled error panel (e.g., `<lightning-icon icon-name="utility:error">`) on error.
- Consistent structure and interaction patterns aligned to project UX standards for title placement, form layout, and Save/Cancel action placement.
- Usage of shared LWC components (e.g., `c-common-header`) and Lightning base components (e.g., `lightning-record-form`) where applicable.
- Event and navigation patterns: custom events with semantic names and typed `detail` payloads; `NavigationMixin` for page navigation; LMS for cross-component messaging.
- Jest tests verifying each state (loading, empty, success, error) and user interaction (button clicks, input changes, event dispatch with correct payloads).
- Apex `*Test.cls` updates for any changed or new Apex methods used by the LWC.

## Working Pattern

1. **Establish State Model**: Define reactive properties (`isLoading`, `hasError`, `isEmpty`) and use them in the template with `<template if:true={isLoading}>` (or `lwc:if`) for conditional rendering.
2. **Reuse Before Building**: Search for existing `c/` namespace components. Use shared components for headers, search inputs, picklists, and multi-selects. If a reusable case exists but no shared component does, consider building one.
3. **Apply Layout Skeleton**: Structure the HTML template as:
   - Header container (SLDS page header or `c-common-header`) with title and key info.
   - Content section with records, fields, or empty-state messaging.
   - Actions section with buttons (Save, Cancel, Refresh, etc.) aligned per UX standards (Save right, Cancel left in footer).
   - Feedback area for toasts or inline error panels.
   - Use SLDS grid classes (`slds-grid`, `slds-var-p-around_medium`, responsive size classes) for spacing and layout.
4. **Data Access Pattern**:
   - Read-only: use `@wire(getRecord, ...)` or `@wire(getListUi, ...)` for LDS caching benefits.
   - Complex queries: write an Apex method and call via `@wire` or imperative import. Ensure the Apex method is bulkified and exceptions are handled in the LWC via `.catch()` or `try/catch`.
   - Create/update: use `lightning-record-edit-form` for simple record edits, or imperative Apex for multi-object transactions.
5. **Implement UI with SLDS**: Use `<lightning-input>`, `<lightning-combobox>`, `<lightning-spinner>`, SLDS utility classes, and design tokens. Avoid inline styles. Use SLDS responsive classes for multi-screen support.
6. **State-Driven Rendering**: Set `this.isLoading = true` at fetch start and `false` on completion. Set `hasError` and capture `errorMessage` on failure. Use `LightningAlert.open()` for critical errors requiring acknowledgement.
7. **Reusable Labels**: Store repeated messages in custom labels or a constants file (e.g., "No records found", "An unexpected error occurred. Please try again or contact support.").
8. **Event Handling and Communication**: Dispatch semantic custom events with `detail` payloads for parent-child communication. Use LMS for sibling/distant components. Use `NavigationMixin` for page navigation. Use `LightningModal`/`LightningAlert` instead of building custom dialogs.
9. **Jest Tests**: Simulate each state by manipulating `@api` properties or mocking wire results. Test button clicks, input changes, and event dispatch with correct payloads. Mock Apex with `jest.mock`. Use `flushPromises()` for async. Abstract `LightningModal`/`LightningAlert` calls into a utility for easier mocking.
10. **Apex Tests**: Test happy path, error cases, boundary conditions, bulk scenarios, and security enforcement (`with sharing`, FLS via `Security.stripInaccessible`).
11. **Incorporate Latest Salesforce UX Features**: Adopt `LightningAlert`, `LightningConfirm`, `LightningPrompt` for dialogs. Use SLDS 2 design tokens and theming hooks (Salesforce Cosmos) for custom branding. Verify component appearance in both classic Lightning Experience and the new opt-in SLDS 2 UI.

## Testing Standard for LWC

**LWC Jest Tests** must cover:

- All visual states: loading (spinner visible), empty (empty message rendered), success (data rendered correctly), error (error message displayed).
- All critical user interactions: Save calls Apex, Cancel navigates or resets, Refresh reloads data.
- Emitted events: assert events are fired with correct `detail` payload for each relevant action.
- Conditional rendering branches and computed properties/getters.
- Use `@salesforce/sfdx-lwc-jest` to stub wire adapters and Apex imports. Use `jest.mock` for Apex and `flushPromises()` for async.

**Apex Test Classes** must cover:

- Happy path: correct data returned or correct DML performed.
- Edge cases: no records found, user lacks permissions, invalid inputs.
- Bulk scenarios: multiple records in parameters to confirm bulk-safe behavior.
- Security: validate `with sharing` behavior and FLS enforcement via `Security.stripInaccessible`.

## Senior Specialist Best Practices (7-10 Years Standard)

- **Design for Reuse**: Extract shared UI and shared JS utilities instead of cloning component logic. Consider a `c/searchBar` LWC for repeated search/filter patterns.
- **Clear Public API**: Expose only necessary data and actions via `@api` properties and custom events. Avoid tight coupling between sibling components.
- **Deterministic Data Flow**: Use `refreshApex` explicitly after mutations rather than relying on implicit cache refresh. Manage all state in tracked properties to keep the rendering model predictable.
- **Error and Exception Handling**: Handle exceptions at every boundary. Log technical details via `console.error` or a logging service; show user-friendly messages in the UI. Standardize error messaging via a shared utility.
- **Naming and Structure**: Use consistent component, event, and method naming (e.g., `accountList`, `accountDetails`, `accountEditor` for related components). Keep folder structure logical and discoverable.
- **Performance**: Debounce search input to limit Apex calls. Use pagination or infinite scroll for large data sets. Avoid unnecessary state changes that trigger rerenders.
- **Stay Updated**: Monitor Salesforce release notes for new LWC base components (e.g., `LightningAlert` in Spring '23, SLDS 2 Cosmos theming). Adopt improvements proactively.
- **Documentation**: Add code comments or a brief description for `@api` properties, events, and any non-obvious implementation decisions (e.g., governor limit constraints).
- **Security**: Use `with sharing` in Apex. Apply `Security.stripInaccessible` for custom Apex returning data to LWC. Do not expose fields the user should not see.

## Definition of Done

- [ ] **Consistent UX structure**: Component layout matches project standards — correct header, spacing (SLDS utility classes or design tokens), and action placement.
- [ ] **State management**: Component cleanly transitions between loading, empty, success, and error states. Each state is tested and user-friendly.
- [ ] **Reusability**: No duplication of existing components or logic. Common functionality uses or contributes to shared components or utilities.
- [ ] **Data handling**: Data fetched and saved using recommended patterns (wired for cacheable reads, imperative Apex for writes). Component refreshes data correctly after mutations.
- [ ] **Accessibility and responsiveness**: Meets accessibility standards (keyboard navigation, screen reader labels). Works on desktop and phone form factors where applicable.
- [ ] **All tests pass**: LWC Jest tests and Apex test classes are added/updated and passing. No existing functionality is broken.
- [ ] **Documentation**: Code comments or description covers usage, `@api` properties, events, and any important implementation constraints.
- [ ] **Consistent look and feel**: Visual integration confirmed against overall app SLDS styling. If Salesforce Cosmos (SLDS 2) is adopted, design tokens and patterns are applied correctly.
