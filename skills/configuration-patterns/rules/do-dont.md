# Do and Don't Guardrails

## Do

- Do use Custom Metadata for deployable business configuration.
- Do use Named Credentials for endpoints and auth.
- Do cache config reads in provider classes.
- Do test missing-config behavior.

## Don't

- Don't hardcode IDs, URLs, credentials, or secrets.
- Don't store secrets in Custom Metadata.
- Don't let callers bypass the provider layer.
- Don't return silent `null` for required config.
