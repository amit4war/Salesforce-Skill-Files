# Configuration Patterns Skill

Production-ready guidance for Custom Metadata, Custom Settings, Named Credentials, Custom Labels, and centralized config providers.

## When To Use

- Replacing hardcoded values
- Adding feature toggles or thresholds
- Defining environment-specific integration settings
- Implementing bypass or operational control flags

## When Not To Use

- Sensitive values would be exposed in source control
- One-off values do not belong in shared project configuration

## Quick Start

1. Classify the setting.
2. Select storage mechanism.
3. Centralize retrieval in a provider.
4. Remove hardcoded callers.
5. Add tests and deployment notes.
