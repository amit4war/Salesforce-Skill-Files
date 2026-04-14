# Agentforce Configuration Evaluator

Use this checklist to validate an Agentforce configuration file. Identify patterns and anti-patterns that might confuse the Atlas Reasoning Engine.

## Agent Definition Rules

### Good Patterns

1. **Clear Agent Name [REQUIRED]**
   - Concise, descriptive, PascalCase convention
   - Example: `SalesforceOpportunityAgent`

2. **Comprehensive Agent Description [REQUIRED]**
   - Clearly states the agent's purpose in 1-2 sentences
   - Outlines core capabilities and domains of expertise

3. **Specific Agent Role Definition [REQUIRED]**
   - Precisely defines the agent's persona and responsibilities
   - Establishes boundaries of capabilities and authority

4. **Detailed Company Context [REQUIRED]**
   - Provides sufficient domain context for the Atlas Reasoning Engine
   - Includes industry, business model, and relevant terminology

### Anti-Patterns

1. **Vague Agent Name [HIGH]** — Generic names that don't indicate purpose (e.g., `SalesAgent`, `Assistant`)
2. **Conflicting Role Definitions [HIGH]** — Contradictory statements about capabilities (e.g., `data analyst who helps with marketing campaigns`)
3. **Overly Broad Capabilities [MEDIUM]** — Suggesting the agent can handle any task without limitations
4. **Missing Company Context [MEDIUM]** — Lacking industry-specific terminology causes the LLM to make assumptions

---

## Topic Definition Rules

### Good Patterns

1. **Clearly Scoped Topics [REQUIRED]** — Focused scope with clear boundaries; distinct from each other with minimal overlap
2. **Descriptive Classification [REQUIRED]** — Classification descriptions clearly distinguish when this topic applies vs. others
3. **Sequential, Specific Instructions [REQUIRED]** — Logical sequence of steps with specific action references and clear inputs/outputs
4. **Error Handling Instructions [RECOMMENDED]** — How to handle common errors or edge cases

### Anti-Patterns

1. **Ambiguous Topic Boundaries [HIGH]** — Topics with unclear boundaries confuse the reasoning engine (e.g., both `Account Management` and `Account Updates` without clear distinction)
2. **Inconsistent Terminology [HIGH]** — Using different terms for the same concept across topics (e.g., `Opportunity` and `Deal` interchangeably)
3. **Circular References [MEDIUM]** — Instructions that reference other topics without clear resolution paths
4. **Missing Decision Criteria [MEDIUM]** — Not providing clear guidance on how to choose between similar actions

---

## Action Definition Rules

### Good Patterns

1. **Specific Action Type [REQUIRED]** — Clearly specify Apex, Flow, Standard Action, etc.
2. **Comprehensive Action Instructions [REQUIRED]** — Detailed explanation of what the action does and when to use it
3. **Complete Input/Output Definitions [REQUIRED]** — All required/optional inputs with data types; all possible outputs including error states
4. **Relationship Context [RECOMMENDED]** — How this action relates to other actions in a workflow

### Anti-Patterns

1. **Undefined Action Types [HIGH]** — Not specifying whether an action is Apex, Flow, REST, etc.
2. **Inconsistent Parameter Naming [HIGH]** — Using different parameter names across similar actions (e.g., `accountName` vs. `acctName`)
3. **Missing Required Input Definitions [HIGH]** — Not clearly marking which inputs are required vs. optional
4. **Incomplete Output Descriptions [MEDIUM]** — Only describing the success case, not how errors are returned

---

## LLM Interaction Guidelines

### Good Patterns

1. **Explicit Reasoning Guidance [RECOMMENDED]** — Provide guidance for multi-step reasoning (e.g., `First determine the account type, then select the appropriate verification method`)
2. **Clear Decision Trees [RECOMMENDED]** — Document decision points with clear criteria for each branch
3. **Scenario-Based Examples [RECOMMENDED]** — Include representative examples of expected inputs and outputs

### Anti-Patterns

1. **Implicit Reasoning Requirements [HIGH]** — Assuming the LLM will infer complex decision logic without guidance
2. **Contradictory Instructions [HIGH]** — Providing conflicting guidance within the same topic or across topics
3. **Ambiguous Prioritization [MEDIUM]** — Not clarifying which goals take precedence when conflicts arise

---

## Validation Checklist

1. **Schema Validation** — All required sections exist (Agent Name, Description, Role, Topics, Actions); consistent formatting
2. **Cross-Reference Validation** — All referenced actions in instructions actually exist in Actions section; topic boundaries are clear and non-overlapping
3. **Clarity and Specificity Check** — No ambiguous or vague directives; all specialized terminology defined
4. **Simulated Query Testing** — Run sample queries against configuration to identify confusion points; test edge cases and ambiguous requests
