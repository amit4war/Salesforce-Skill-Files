# Tech Stack and Constraints

- Lightning Web Components
- SLDS (latest project-supported version)
- Lightning Data Service + UI API wire adapters
- Apex for complex business operations
- Lightning Message Service and NavigationMixin
- Jest (`@salesforce/sfdx-lwc-jest`)

## Constraints

- Prefer base `lightning-*` components.
- Keep custom CSS minimal and SLDS-compatible.
- Use secure Apex patterns (sharing + CRUD/FLS handling).
