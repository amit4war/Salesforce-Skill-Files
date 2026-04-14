---
name: create-agentforce-config
description: Create and configure Salesforce Agentforce agent configuration files including topics, actions, variables, and filters. Use when the user wants to create a new Agentforce agent, define agent topics and actions, or build an agent configuration markdown file.
---

# Create Agentforce Configuration

## Workflow

Follow these steps when creating a new Agentforce agent configuration.

### Step 1: Gather Requirements

Ask the user for:
- **Agent Name** (PascalCase, descriptive)
- **Agent Purpose** (what problem does it solve?)
- **Company Context** (industry, terminology)
- **Key Use Cases** (what should the agent handle?)

### Step 2: Draft the Configuration

Use the structure defined in [template.md](template.md). **Do not invent your own format.**

Fill in:
1. Agent Name, Description (max 255 chars), Role (max 255 chars), Company Description (max 255 chars)
2. A **Global_Instructions** topic (mandatory for every agent — see guidelines below)
3. Business-specific topics based on user requirements

### Step 3: Select Actions

**CRITICAL — Standard Actions First:**
- Read [standard-actions.md](standard-actions.md) and search for actions that match each requirement.
- If a standard action exists, use it. Include its name, purpose, and documentation link.
- Only create a custom action (Flow, Apex, etc.) when no standard action fits. If creating a custom action, document why standard actions were insufficient.

### Step 4: Apply Guidelines

Read [guidelines.md](guidelines.md) for detailed rules on:
- **Global Instructions Topic** — required structure and content
- **Action format** — inputs, outputs, error handling, standard vs. custom justification
- **Variables & Filters** — authentication gates, sequential process controls, data dependencies
- **Response Grounding** — citation requirements and data source configuration

### Step 5: Validate

Read [evaluator.md](evaluator.md) and check the configuration against:
- Agent definition patterns (clear name, role, description, company context)
- Topic definition patterns (scoped topics, sequential instructions, error handling)
- Action definition patterns (typed actions, complete I/O, relationship context)
- Cross-reference validation (all referenced actions exist, no overlapping topics)

## Reference Files

| File | Purpose |
|------|---------|
| [template.md](template.md) | Output format — use this structure for every config file |
| [standard-actions.md](standard-actions.md) | 200+ Salesforce standard actions — search here first |
| [guidelines.md](guidelines.md) | Detailed rules for topics, actions, variables, filters, grounding |
| [evaluator.md](evaluator.md) | Validation patterns and anti-patterns |
