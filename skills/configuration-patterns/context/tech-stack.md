# Tech Stack and Constraints

- Custom Metadata Types
- Hierarchy Custom Settings
- Named Credentials and External Credentials
- Custom Labels
- Apex configuration provider classes

## Constraints

- Secrets must not live in source-controlled metadata
- Callers should not query config directly
- Missing configuration must fail safely
