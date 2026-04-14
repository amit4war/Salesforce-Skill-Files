# Do and Don't

## Do

- Use SLDS-consistent layouts and base components.
- Implement loading, empty, success, and error states.
- Reuse existing `c/*` shared components before creating new ones.
- Use `NavigationMixin` and LMS for standard communication/navigation.
- Add Jest coverage for core UI behavior.

## Don’t

- Don’t hardcode styles that conflict with SLDS.
- Don’t put complex business logic in the component when Apex/service layers are required.
- Don’t skip accessibility (labels, keyboard flow, focus handling).
- Don’t expose sensitive data in UI errors or logs.
