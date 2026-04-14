# SOQL and Selectors Skill

Production guide for centralized, secure, and reusable SOQL access using selector classes.

## When To Use

- Creating new query methods
- Replacing duplicated inline SOQL
- Hardening query security or performance

## When Not To Use

- Query is a one-off script outside app codebase
- Requirement is unclear on object/filter/security context

## Quick Start

1. Define object, required fields, and filters.
2. Design selector method signature for bulk use.
3. Apply security strategy and sharing mode.
4. Update call sites to selector usage.
5. Add selector tests for normal and edge behavior.
