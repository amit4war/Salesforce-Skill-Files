# Agentforce Configuration Guidelines

## Global Instructions Topic

### Mandatory Global Instructions Topic
Every Agentforce agent configuration must include a Global Instructions topic that establishes universal conversation guidelines. This topic ensures consistent behavior across all agent interactions.

#### Global Instructions Topic Structure
```markdown
### Global_Instructions

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
Always ask a follow up question to keep the conversation going until the user has no more questions.
```

#### Global Instructions Requirements
- **No Actions Required:** This topic does not require any associated actions
- **Universal Application:** These instructions apply to all conversations regardless of topic
- **Customization:** Additional instructions can be added based on specific business requirements
- **Brand Consistency:** Maintains consistent tone and behavior across all agent interactions

---

## Standard Actions First Policy

- **CRITICAL:** Always prioritize Salesforce standard actions over custom actions when available
- Before creating any custom action, search `standard-actions.md` for suitable existing actions
- Only create custom actions when no standard action meets the specific requirement

### Standard vs Custom Action Decision Matrix

**Use Standard Action When:**
- Exact functionality match exists
- Standard action covers 80%+ of requirements
- Minor configuration can bridge gaps

**Create Custom Action When:**
- No standard action addresses the core requirement
- Business logic is highly specific to organization
- Standard action lacks critical parameters or outputs

### Action Selection Process
1. Search `standard-actions.md` for existing actions that match the requirement
2. If standard action exists, use it with appropriate configuration
3. Only if no standard action fits, design a custom Flow or Apex action
4. Document why standard actions were not suitable (if creating custom)

### Custom Action Justification Template
When creating custom actions, always include:

```markdown
**Custom Action Justification:**
- **Standard Actions Evaluated:** [List of standard actions considered]
- **Why Standard Actions Don't Fit:** [Specific reasons why standard actions are insufficient]
- **Custom Requirements:** [Specific business logic that requires custom implementation]
```

---

## Action Best Practices

- **Naming:** Use descriptive, action-oriented names that clearly indicate what the action does
- **Instructions:** Provide clear, step-by-step instructions for the action
- **Parameters:** Define all required and optional inputs with clear descriptions
- **Error Handling:** Include guidance for common error scenarios

### Action Validation Checklist
Before finalizing any action configuration, verify:
- [ ] Searched `standard-actions.md` for existing solutions
- [ ] Evaluated if standard action meets core requirements
- [ ] Documented justification if choosing custom over standard
- [ ] Confirmed standard action has necessary parameters and outputs
- [ ] Reviewed Salesforce documentation for standard action limitations

---

## Variables and Filters

### Variables Overview
Variables store information during agent conversations and enable sophisticated state management.

#### Variable Types
- **Context Variables**: Derived from messaging session (immutable during session)
  - Standard/custom fields from MessagingSession object
  - Pre-chat form data
  - Previous session information
- **Custom Variables**: Store action outputs and conversation state (mutable)
  - Track authentication status
  - Store action outputs for reuse
  - Create dependencies between actions

#### Variable Naming Standards
- Use camelCase: `customerAccountId`, `authenticationStatus`
- Be descriptive: `orderNumber` not `orderNum`
- Include data type context: `isAuthenticated` (boolean concept), `customerScore` (numeric)
- Data Types: Currently support Text and Number types only

### Filters Overview
Filters provide deterministic control over topic and action availability during reasoning. Unlike instructions, filters operate at the system level to completely remove or include topics and actions.

#### When to Use Filters vs Instructions
- **Use Filters For**: 
  - Security controls (authentication requirements)
  - Sequential workflow enforcement (Step A must complete before Step B)
  - Data-driven availability (action only available if specific data exists)
- **Use Instructions For**:
  - Conversation flow guidance
  - Response formatting
  - Business context explanation

#### Filter Implementation
- **Topic-Level Filters**: Control entire topic availability
  - Example: `authenticationStatus = "verified"` for Account Management topic
- **Action-Level Filters**: Control specific action availability within a topic
  - Example: `orderNumber != null` for Order Status actions

#### Common Filter Patterns
- **Authentication**: `authenticationStatus = "verified"`
- **Data Dependency**: `recordId != null`
- **User Type**: `userType = "premium"`
- **Sequential Process**: `stepOneComplete = "true"`

---

## Response Grounding and Citations

### Grounding Requirements
The Atlas Reasoning Engine ensures responses are grounded in source information:
- **Data Sources:** CRM records, knowledge articles, external documents, previous conversation context, action outputs.
- **Grounding Process:** Validates response content against sources, ensures no hallucinated information, checks for prompt injection risks.
- **Citation Requirements:** Include source references when using knowledge, link to relevant documentation, reference specific records or articles.

### Best Practices for Grounded Responses
- Use RAG-enabled actions for knowledge queries.
- Configure proper data source access and retrievers.
- Specify required citation formats in topic instructions.
- Define fallback responses and escalation procedures.
- Include confidence indicators for responses.

---

## General Best Practices
- **Completeness:** Ensure topics cover the full user journey and that all necessary actions are defined.
- **Clarity:** Use clear, user-centric language in all descriptions and instructions.
- **Consistency:** Maintain consistent formatting and naming conventions throughout the file.
- **Error Handling:** Define how to handle errors and edge cases within topic instructions and action definitions.
- **Action-Topic Mapping:** Ensure each action mentioned in topic instructions is defined in the Actions section.
- **Testing:** Test your configuration against sample queries before deployment.

---

## Reference Documentation

### Official Salesforce Documentation
- **Einstein Documentation:** https://developer.salesforce.com/docs/einstein/genai/guide/get-started-agents.html
- **Agentforce Documentation:** https://help.salesforce.com/s/articleView?id=ai.generative_ai.htm&type=5
- **Topic Instructions Best Practices:** https://help.salesforce.com/s/articleView?id=ai.copilot_topics_instructions.htm&type=5
- **Action Instructions Best Practices:** https://help.salesforce.com/s/articleView?id=ai.copilot_actions_instructions.htm&type=5
- **Troubleshooting Agents:** https://help.salesforce.com/s/articleView?id=ai.copilot_troubleshoot.htm&type=5
- **Agentforce Variables:** 
  - https://developer.salesforce.com/blogs/2024/06/agentforce-variables-context-for-every-conversation
  - https://help.salesforce.com/s/articleView?id=ai.copilot_variables.htm&type=5
- **Grounding and Citations:** https://help.salesforce.com/s/articleView?id=ai.copilot_grounding.htm&type=5
