# Project Structure Standard

This repository is organized as a shared Salesforce DX delivery accelerator for multi-developer teams.

## Standard Repository Layout

```text
.
├─ .ai/                         # Tool-agnostic AI context, skills, and prompts
├─ .claude/                     # Claude-specific reusable command playbooks
├─ .github/                     # Copilot instructions, PR templates, scoped rules
├─ .vscode/                     # Shared editor recommendations and workspace settings
├─ config/                      # Scratch org and environment templates
├─ docs/                        # Delivery, architecture, and governance documentation
├─ force-app/                   # Salesforce DX source-format metadata
│  └─ main/default/
│     ├─ classes/
│     ├─ customMetadata/
│     ├─ lwc/
│     ├─ objects/
│     ├─ permissionsets/
│     └─ triggers/
├─ manifest/                    # Package manifests for targeted retrieve/deploy flows
├─ scripts/                     # Reusable local scripts and anonymous Apex helpers
└─ skills/                      # Cross-tool Salesforce development skills
```

## Folder Intent

### `.ai/`
Shared AI knowledge base for all contributors regardless of assistant. Store patterns, decision context, and assistant-neutral working guidance here.

### `.claude/` and `.github/`
Assistant-specific layers that point back to the canonical rules in [docs/ai/baseline-salesforce-rules.md](docs/ai/baseline-salesforce-rules.md).

### `config/`
Holds environment templates such as scratch org definitions. Keep environment-specific secrets out of this folder.

### `docs/`
Client-ready documentation for onboarding, standards, architecture, and workflow.

### `force-app/main/default/`
Primary deployable metadata root. All Salesforce source intended for deployment should live here unless packaging strategy changes.

### `manifest/`
Use for package manifests that support targeted retrieval or controlled validation.

### `scripts/`
Utility scripts, anonymous Apex, or repeatable local helper assets. Keep them reusable and documented.

### `skills/`
Reusable playbooks for AI-assisted development tasks such as triggers, testing, selectors, deployment, and LWC data patterns.

## Team Rules for Adding Content
- Add new deployable metadata only under the agreed Salesforce DX source path.
- Add new documentation under `docs/` unless it is assistant-specific.
- Add reusable AI patterns under `.ai/skills/` and mirror concise versions under `skills/` when helpful.
- Do not create developer-personal folders in the repository.
- Do not commit machine-local settings, auth files, or generated noise.

## Merge-Friendly Naming Guidance
- Use descriptive, stable API names.
- Keep one concern per file where platform conventions allow it.
- Prefer small, focused changes over large mixed commits.
- Update adjacent docs when introducing a new pattern, module, or shared dependency.
