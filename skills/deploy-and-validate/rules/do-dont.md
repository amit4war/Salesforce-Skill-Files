# Do and Don't

## Do

- Confirm target org alias before every command.
- Validate before deploying medium/high-risk changes.
- Use explicit test-level arguments.
- Keep manifest scoped to changed components.
- Capture and share deploy job IDs.

## Don’t

- Don’t deploy with unknown target alias.
- Don’t use `--ignore-errors` or `--ignore-warnings` as shortcuts.
- Don’t run destructive changes without impact review and rollback notes.
- Don’t use `NoTestRun` in production.

## Security and Risk Notes

- Treat profile/permission/sharing changes as high risk by default.
- Never expose secrets, tokens, or auth material in logs.
