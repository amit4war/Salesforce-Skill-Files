
Salesforce Skill Accelerator
A shared Salesforce development accelerator providing reusable patterns, skills, and best practices for building scalable, secure, and maintainable Salesforce applications. This repository serves as a team knowledge base and starting point for common enterprise patterns in Apex, LWC, Flows, and integrations.

🚀 Features
Apex Patterns: Trigger-handler, service, selector, and async patterns with bulk-safe implementations.
Testing Framework: Deterministic test data factories and comprehensive test coverage strategies.
Configuration Management: Metadata-driven settings using Custom Metadata Types, Custom Settings, and Named Credentials.
Deployment & Validation: CLI-based workflows for sandbox-first development and production deployments.
LWC Best Practices: Data patterns, component composition, and Lightning Design System integration.
SOQL Optimization: Selector classes and query performance guidelines.
Debugging & Code Review: Root cause analysis tools and static analysis rules (SonarQube, PMD).
Project Standards: Naming conventions, security guardrails, and bypass mechanisms.
📁 Project Structure

.├── CLAUDE.md                    # Project intent and workflow overview├── CONTRIBUTING.md              # Contribution guidelines├── package.json                 # Node.js dependencies (if any)├── sfdx-project.json            # Salesforce project configuration├── config/                      # Scratch org definitions├── docs/                        # Documentation│   ├── ai/                      # AI baseline rules│   ├── development-workflow.md  # Branching, PR, and deployment flow│   └── project-structure.md     # Detailed folder explanations├── force-app/                   # Salesforce metadata (DX format)│   └── main/default/            # Source code and metadata│       ├── classes/             # Apex classes│       ├── customMetadata/      # Custom Metadata Types│       ├── lwc/                 # Lightning Web Components│       ├── objects/             # Custom objects and fields│       ├── permissionsets/      # Permission sets│       └── triggers/            # Apex triggers├── manifest/                    # Package manifests├── scripts/                     # Utility scripts│   └── apex/                    # Apex-specific scripts└── skills/                      # Domain-specific skill guides    ├── apex-trigger-pattern/    # Trigger-handler patterns    ├── async-apex-patterns/     # Batch, Queueable, Schedulable    ├── configuration-patterns/  # Metadata-driven config    ├── debugging-and-code-review/ # Analysis and review tools    ├── deploy-and-validate/     # CLI deployment workflows    ├── lwc-data-patterns/       # LWC development patterns    ├── soql-and-selectors/      # Query optimization    └── testing-and-testdata/    # Test strategies and factories
🛠️ Getting Started
Prerequisites
Salesforce CLI (sf) installed and authenticated.
Node.js (for any scripts or tooling).
Git for version control.
A Salesforce Developer Edition or sandbox org.
Setup
Clone the repository:


git clone https://github.com/amit4war/Salesforce-Skill-Files.gitcd Salesforce-Skill-Files
Install dependencies (if any):


npm install
Create a scratch org (for development):


sf org create scratch --definition-file config/project-scratch-def.json --alias MyScratchOrg --set-default
Push source to scratch org:


sf project deploy start --target-org MyScratchOrg
Open the org:


sf org open --target-org MyScratchOrg
For detailed setup, see development-workflow.md.

📖 Usage
Applying Skills
Each skills folder contains a SKILL.md file with:

Purpose: When and why to use the pattern.
Guardrails: Rules and best practices.
Working Pattern: Step-by-step implementation guide.
Examples: Code snippets and templates.
Start with the skill that matches your task:

New Apex logic? → testing-and-testdata/SKILL.md
Adding configuration? → configuration-patterns/SKILL.md
Deploying changes? → deploy-and-validate/SKILL.md
Development Workflow
Sandbox-first: Develop and test in scratch orgs or sandboxes.
Feature branches: Create feature/<work-item>-<description> branches.
Pull requests: Include validation commands, test results, and risk summaries.
Validation: Use sf project deploy validate before merging.
Deployment: Run sf project deploy start with appropriate test levels.
See development-workflow.md for full details.

Key Commands
Validate deployment: sf project deploy validate --source-dir force-app --test-level RunLocalTests --target-org <alias>
Deploy: sf project deploy start --source-dir force-app --test-level RunLocalTests --target-org <alias>
Run tests: sf apex run test --target-org <alias> --test-level RunLocalTests --code-coverage
🤝 Contributing
We welcome contributions! Follow these steps:

Read the guidelines: See CONTRIBUTING.md and development-workflow.md.
Choose a skill: Identify the relevant skills folder or create a new one if needed.
Follow standards: Adhere to naming conventions, security rules, and the Definition of Done in each SKILL.md.
Test thoroughly: Ensure ≥85% coverage and all guardrails are met.
Submit a PR: Include validation output, test results, and a clear description.
Project Standards
Naming: PascalCase for fields/objects, camelCase for methods/variables (with Hungarian prefixes for Apex).
Security: with sharing, FLS enforcement, no hardcoded secrets.
Code Quality: Bulk-safe, no SOQL/DML in loops, proper error handling.
Bypass Mechanism: All triggers and flows check BypassSettings__c flags.
API Version: Target 62.0 for all metadata.
📋 Definition of Done (All Tasks)
 Code compiles with no errors/warnings.
 ≥85% test coverage with meaningful assertions.
 No hardcoded IDs, URLs, or credentials.
 Naming conventions followed.
 Bypass mechanism implemented where applicable.
 Documentation updated (if patterns change).
 All existing tests pass.
📄 License
This project is licensed under the MIT License — see LICENSE for details (add one if not present).

🆘 Support
Issues: Open a GitHub issue for bugs or feature requests.
Discussions: Use GitHub Discussions for questions or pattern clarifications.
Docs: Refer to docs for detailed guides.
Happy coding! This accelerator is designed to evolve with your team's needs — contribute back to keep it sharp. 🚀
