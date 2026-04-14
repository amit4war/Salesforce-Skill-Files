# Agentforce Agent Configuration Template

Use this exact structure when generating an Agentforce agent configuration file.

## Important Documentation References
- [Topic Instructions Best Practices](https://help.salesforce.com/s/articleView?id=ai.copilot_topics_instructions.htm&type=5)
- [Action Best Practices](https://help.salesforce.com/s/articleView?id=ai.copilot_actions_instructions.htm&type=5)

## Template

```markdown
# Agentforce Agent Configuration for [SPECIFIC PURPOSE]

## Agent Name
**[AGENT NAME]**

## Agent Description
[CONCISE DESCRIPTION OF AGENT'S PURPOSE AND CAPABILITIES - MAX 255 CHARACTERS]

## Agent Role
You are [DETAILED DESCRIPTION OF HOW THE AGENT ASSISTS USERS - MAX 255 CHARACTERS]

## Company Description
[BRIEF DESCRIPTION OF THE COMPANY AND ITS RELEVANT CONTEXT - MAX 255 CHARACTERS]

## Topics

### 1. Global_Instructions

**Classification Description:**
System Level Prompt.

**Scope:**
Global Level of Instructions.

**Instructions:**
All answers should comply with these guidelines:
1. Clear and Accurate: Use simple, straightforward language and provide correct, reliable information.
2. Concise and to the point: Avoid a preamble in your responses. When answering a question you provide information succinctly, without unnecessary elaboration.
3. Empathetic and Polite: Show understanding and respect, using a friendly and supporting tone.
4. Avoid apologies like "I am sorry."

**Additional Optional Instructions:**
[ADD BUSINESS-SPECIFIC GLOBAL INSTRUCTIONS HERE]

### 2. [TOPIC NAME]

**Classification Description:**
[BRIEF DESCRIPTION OF WHAT THIS TOPIC HANDLES]

**Scope:**
The agent's job is to [DESCRIPTION OF THE AGENT'S RESPONSIBILITIES FOR THIS TOPIC]

**Instructions:**
1. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
2. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
3. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
4. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
5. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
6. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
7. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
8. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
9. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
10. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]

### 3. [TOPIC NAME]

**Classification Description:**
[BRIEF DESCRIPTION OF WHAT THIS TOPIC HANDLES]

**Scope:**
The agent's job is to [DESCRIPTION OF THE AGENT'S RESPONSIBILITIES FOR THIS TOPIC]

**Instructions:**
1. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
2. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
3. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
4. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
5. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
6. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
7. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
8. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
9. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]
10. [NUMBERED INSTRUCTION - FOLLOW TOPIC INSTRUCTIONS BEST PRACTICES]

## Actions

### 1. [ACTION NAME]

**Action Type:** [Standard Action | Flow | Apex | Prompt Template | MuleSoft API | External Service | Predictive Model]

**Standard Action Reference:** (If using standard action)
- **Standard Action Name:** [Name from standard-actions.md]
- **Purpose:** [Why this standard action was selected]
- **Documentation:** [Link to Salesforce documentation]

**Action Instructions:**
[DESCRIPTION OF WHAT THIS ACTION DOES AND WHEN TO USE IT]

**Inputs:**
- [INPUT NAME] (Required): [DESCRIPTION OF INPUT]
- [INPUT NAME] (Optional): [DESCRIPTION OF INPUT]

**Outputs:**
- [OUTPUT NAME]: [DESCRIPTION OF OUTPUT]
- [OUTPUT NAME]: [DESCRIPTION OF OUTPUT]

**Error Handling:**
[HOW THE ACTION HANDLES COMMON ERROR SCENARIOS]

### 2. [ACTION NAME]

**Action Type:** [Standard Action | Flow | Apex | Prompt Template | MuleSoft API | External Service | Predictive Model]

**Standard Action Reference:** (If using standard action)
- **Standard Action Name:** [Name from standard-actions.md]
- **Purpose:** [Why this standard action was selected]
- **Documentation:** [Link to Salesforce documentation]

**Custom Action Justification:** (If NOT using standard action)
- **Standard Actions Evaluated:** [List of standard actions considered]
- **Why Standard Actions Don't Fit:** [Specific reasons]
- **Custom Requirements:** [Specific business logic that requires custom implementation]

**Action Instructions:**
[DESCRIPTION OF WHAT THIS ACTION DOES AND WHEN TO USE IT]

**Inputs:**
- [INPUT NAME] (Required): [DESCRIPTION OF INPUT]
- [INPUT NAME] (Optional): [DESCRIPTION OF INPUT]

**Outputs:**
- [OUTPUT NAME]: [DESCRIPTION OF OUTPUT]
- [OUTPUT NAME]: [DESCRIPTION OF OUTPUT]

**Error Handling:**
[HOW THE ACTION HANDLES COMMON ERROR SCENARIOS]
```
