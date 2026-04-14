# LWC Data Patterns Skill

Production guide for building Salesforce LWCs with consistent UX, robust state management, and secure data access.

## When To Use

- New LWC development
- LWC refactor for maintainability
- Standardizing loading/error/empty/success states
- Aligning component architecture across team

## When Not To Use

- Task is pure Apex with no UI impact
- No clarity on component surface, data source, or user actions

## Quick Start

1. Define component purpose and user journey.
2. Select data strategy: `wire`, LDS, or Apex.
3. Define visual states and action buttons.
4. Reuse shared components first.
5. Implement and test UI + Apex paths.

## Deliverables

- Updated `.html`, `.js`, `.js-meta.xml` (and `.css` only if needed)
- Clear state handling for loading/empty/success/error
- Reusable architecture and event contracts
- Jest tests + Apex tests where applicable
