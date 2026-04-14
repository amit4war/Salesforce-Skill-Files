# Authoring Rules

## Writing Style

- Use short, imperative instructions.
- Prefer exact steps over descriptive prose.
- Define decision points explicitly (`if/then` style).

## Code Change Standards

- Keep trigger files logic-free.
- Keep methods single-purpose and event-aware.
- Use clear names aligned with object and business behavior.
- Keep cross-layer dependencies explicit and minimal.
- Every `public` and `global` class, method, and property must include an ApexDoc comment block with `@description`, `@param`, and `@return` tags. This is a Medium SonarQube rule that will be flagged on every deployment scan.

## Documentation Standards

- Every task output must include:
  - What changed
  - Why it changed
  - How it was validated
  - Known risks and rollback notes
- Use checklists for acceptance and review gates.

## Task Decomposition Rules

For complex work, split into:

1. Discovery (existing automation and constraints)
2. Design (event mapping and layer boundaries)
3. Implementation (trigger/handler/domain/service/selector)
4. Testing (targeted + bulk + edge)
5. Deployment readiness (risk and rollback)

## Output Format Rules

Final skill-driven response should include:

1. Scope summary
2. Files changed
3. Implementation notes by layer
4. Test evidence
5. Deployment and rollback guidance
