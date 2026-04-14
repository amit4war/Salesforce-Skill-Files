---
description: Project-wide Salesforce development standards. Always apply these rules to every task — naming conventions, Apex coding, SOQL, sharing/security, test classes, LWC, Flow, OmniStudio, and bypass mechanism. Load alongside any domain skill.
globs: "**/*"
---

# Project Guidelines — Salesforce Development Standards

These rules apply to **every task** in this project, regardless of the feature or domain being worked on. All generated code, configuration, and metadata must comply with every section below.

---

## 1. Metadata Naming Standards

### Fields

- All field API names **MUST** be PascalCase (e.g., `BookedCapacity__c`).
- Field names **SHOULD NOT** contain underscores except for the required `__c` suffix.
- Every field **MUST** have a meaningful description.
- If the field purpose is ambiguous, help text **MUST** be defined. Help text **COULD** also be added on clear fields for extra clarity.
- Custom field names and labels must be unique for their object. Labels are determined by business need.

### Custom Objects

- Custom object API names must be unique org-wide and use the `__c` suffix (e.g., `DailyCapacity__c`).

### Profiles

- Create a Base Custom Profile with minimum access only.
- No extra permissions at the Profile level.
- Profile name: `Regulatory User Profile`

### Permission Sets

- Name format: `<Object Name> Permissions`
- Example: `Account Permissions`

### Sharing Rules

- Format: `Share[Object][Recipient]`
- Examples: `ShareAllocationRecordsToSupplyChain`, `ShareSiteRecordsToSlotManagementUsers`

---

## 2. Apex Naming Conventions

### Class Names

| Type                 | Convention                    |
| -------------------- | ----------------------------- |
| Controller           | `<VFPageName>Controller`      |
| Controller Extension | `<VFPageName>Extension`       |
| Batch                | `<Functionality>Batch`        |
| Schedulable          | `<Functionality>Scheduleable` |
| Queueable            | `<Functionality>Queueable`    |
| Utility              | `<Functionality>Utility`      |
| Trigger              | `<SObject>Trigger`            |
| Trigger Handler      | `<SObject>TriggerHandler`     |
| Service              | `<Functionality>Service`      |
| Domain               | `<Functionality>Domain`       |
| Selector             | `<SObject>Selector`           |
| Interface Service    | `INT_<Functionality>Service`  |
| Interface Utility    | `INT_<Functionality>Utility`  |
| Test                 | `<ClassName>Test`             |
| Exception            | `<Functionality>Exception`    |
| Other                | `<Functionality>`             |

### Method Names

- camelCase always.
- Name must articulate the method's purpose and include the return type where meaningful.
- Example: `getContactsByName`

### Variable Names — Hungarian Notation Prefix

| Type     | Prefix   | Example        |
| -------- | -------- | -------------- |
| Blob     | `bin`    | `binImage`     |
| Boolean  | `b`      | `bStatus`      |
| Date     | `dt`     | `dtStartDate`  |
| DateTime | `dttm`   | `dttmStart`    |
| Decimal  | `d`      | `dValue`       |
| Double   | `dbl`    | `dblWidth`     |
| ID       | _(none)_ | `accountId`    |
| Integer  | `i`      | `iLoop`        |
| Long     | `lng`    | `lngTotal`     |
| String   | `s`      | `sAccountName` |
| Time     | `tm`     | `tmStartTime`  |

- camelCase, lowercase first letter.
- Must not start with `_` or `$`.
- Short, meaningful, and mnemonic.

### Apex Data Structure Prefixes

| Type    | Prefix | Example                              |
| ------- | ------ | ------------------------------------ |
| List    | `lst`  | `lstContacts`, `lstContactsToUpdate` |
| Map     | `map`  | `mapIdContact`                       |
| Set     | `set`  | `setAccountName`                     |
| sObject | `obj`  | `objAccount`                         |
| Enum    | `enu`  | `enuSeason`                          |

---

## 3. Base Coding Rules — All code must satisfy every item

- Compiles with no errors or warnings.
- Committed to approved source control. Commits are atomic, frequent, descriptive, pushed daily.
- Naming conventions strictly followed.
- **Never** store usernames, passwords, Salesforce IDs, or URLs in code or source control.
- **≥85% code coverage** with proper `System.assert` statements.
- Always use the latest generally available Salesforce API version. Updated classes must be saved to current API version.
- Proper Error and Execution Handling in all code.
- All triggers must be bulkified.
- Prefer Configuration over Customization.
- Reduce cyclomatic complexity.
- Always perform bulk DML on collections — never per-record.
- Use `Database.<dml>` for partial record processing.
- **Do not perform DML inside loops.**
- Use `try-catch` blocks with an error logging mechanism for all caught exceptions.
- Use `Database.setSavepoint` and `Database.rollback` when mixing DML and Callouts.

---

## 4. SOQL Standards

- SOQL keywords **MUST** be ALL CAPS: `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`, `TODAY`, `GROUP BY`, etc.
- Break SOQL across multiple lines — each clause on its own line.
- List fields in ascending alphabetical order within `SELECT`.
- Each sub-query on its own line.
- **Always use bind variables** — never string-concatenate in WHERE clauses.
- If dynamic SOQL is required, always escape with `String.escapeSingleQuotes()`:

```apex
String strQuery = 'SELECT Id, OwnerId, Status, Subject'
    + ' FROM Task'
    + ' WHERE IsClosed = false'
    + ' AND CampaignID__c = \'' + String.escapeSingleQuotes(strCampaignID) + '\''
    + ' LIMIT 400';
Database.query(strQuery);
```

---

## 5. Sharing and Security

- Use `with sharing` to respect record access. Apex controllers **must** be `with sharing`.
- When using `with sharing`, always check `isCreateable()`, `isUpdateable()`, `isDeletable()` before DML.
- A class inherits the sharing context of its caller unless explicitly declared otherwise.
- Enforce FLS using the `Schema` class — Apex runs in System mode by default.
- Any `without sharing` usage must be documented with a clear business justification comment.
- Reference: [Enforcing CRUD and FLS](https://developer.salesforce.com/page/Enforcing_CRUD_and_FLS)

---

## 6. Test Class Standards

- Separate files with `@isTest` annotation (does not count toward org code size limits).
- Cover all use cases: positive, negative, bulk (≥20 records), and single record.
- Use `System.runAs()` to test different user contexts.
- Create all test data within the test class using `@TestSetup` or a `TestDataFactory`. Never rely on org data.
- Create all test data **before** `Test.startTest()`.
- Only code under test goes inside `Test.startTest()` / `Test.stopTest()`.
- Meaningful assertions required: `System.assert()`, `System.assertEquals()`, `System.assertNotEquals()`.

---

## 7. LWC Standards

### Naming

- Component folder: camelCase `<Functionality><TypeOfComponent>` → renders as kebab-case tag.
  - Example: `detailedCapacityTable` → `<c-detailed-capacity-table>`
- JavaScript properties: camelCase. HTML attributes: kebab-case.

### HTML & JS Rules

- Always prefer OOTB Lightning base components (`lightning-*`).
- Use Lightning Design System. Override only where necessary.
- **Never use `!important` in CSS** — use specificity instead.
- Use `lwc:if` for conditional rendering.
- Prefer `@wire` to a property for predictable `.data` / `.error` pattern.
- Use composition to keep components small and maintainable.
- Use `this.querySelector()` / `this.querySelectorAll()` — not `this.template.querySelector()`.
- Use `Element.getElementsByTagName()` / `Element.getElementsByClassName()` on parent elements where supported.

### Apex from LWC

| Scenario                                              | Approach                                                                |
| ----------------------------------------------------- | ----------------------------------------------------------------------- |
| Read metadata or rarely-changing data                 | `@wire` (cached, performance-optimized)                                 |
| DML required or result must be non-read-only          | Imperative Apex (`cacheable=false`)                                     |
| Invoked on a specific user event (e.g., button click) | Imperative Apex                                                         |
| Stale cache needs refresh                             | `refreshApex()`                                                         |
| Single record create / update / delete                | Lightning Data Service (`createRecord`, `updateRecord`, `deleteRecord`) |

- LDS preferred for single-record CRUD — maintains local cache consistency, no Apex required.
- You cannot call `refreshApex()` on data fetched imperatively.

---

## 8. Flow Standards

### Flow Types

| Type                             | Use Case                                                         |
| -------------------------------- | ---------------------------------------------------------------- |
| Before Save (Record-Triggered)   | Same-record field updates before save                            |
| After Save (Record-Triggered)    | Related record updates, email alerts, post-insert/update actions |
| Before Delete (Record-Triggered) | Actions or cleanup before record deletion                        |
| Screen Flow                      | Step-by-step user interaction to gather and manipulate data      |
| Scheduled Flow                   | Scheduled batch processing — single-use, daily, or weekly        |

### Flow Naming

| Type                | Format                           | Example                                         |
| ------------------- | -------------------------------- | ----------------------------------------------- |
| Before Save         | `ObjectAPI-BeforeSave-Process`   | `Account-BeforeSave-SetSiteID`                  |
| After Save          | `ObjectAPI-AfterSave-Process`    | `Account-AfterSave-SendSiteCreationEmail`       |
| Before Delete       | `ObjectAPI-BeforeDelete-Process` | `Account-BeforeDelete-DeactivateSuiteApprovals` |
| Screen Flow         | `Screenflow-Feature`             | `Screenflow-CreateCapacityConversion`           |
| Scheduled (object)  | `Scheduled-ObjectAPI-Feature`    | `Scheduled-Account-UpdateActiveFlag`            |
| Scheduled (general) | `Scheduled-Feature`              | `Scheduled-UpdateRecords`                       |

_(Exclude `__c` from object API names in flow names.)_

### Trigger Orders

- All Record-Triggered flows **MUST** have an explicit Trigger Order — never leave at default.
- Set in increments of 5 (Flow 1 = 1, Flow 2 = 5, Flow 3 = 10) to allow future insertion.
- Create Only flows must run before Create+Update or Update Only flows.

### Flow Variable Naming

| Type                  | Convention                    | Example                                    |
| --------------------- | ----------------------------- | ------------------------------------------ |
| Screen flow input     | `recordId`                    | `recordId`                                 |
| Record variable       | `r_ObjectAPI`                 | `r_Account`, `r_Account_Clone`             |
| Record collection     | `rColl_ObjectAPI`             | `rColl_Account`, `rColl_Account_ForCreate` |
| Non-record variable   | `v_Purpose`                   | `v_DailyCapacityDate`                      |
| Non-record collection | `vColl_Purpose`               | `vColl_DailyCapacityIDs`                   |
| Formula               | `f_Purpose`                   | `f_Today`, `f_RemainingSourceCapacity`     |
| Choice                | `ch_Value`                    | `ch_Yes`, `ch_Other`                       |
| Collection Choice Set | `chSetColl_Purpose`           | `chSetColl_SelectedSuites`                 |
| Picklist Choice Set   | `chSet_ObjectAPI_Field`       | `chSet_Account_DemandType`                 |
| Record Choice Set     | `chSetRecs_ObjectAPI_Purpose` | `chSetRec_Suite_SiteFiltered`              |
| Text Template         | `tt_Purpose`                  | `tt_SiteNotificationBody`                  |
| Constant              | `c_Purpose`                   | `c_Pi`                                     |

### Flow Element Naming

| Element        | Label Format            | API Name Format         | Example                              |
| -------------- | ----------------------- | ----------------------- | ------------------------------------ |
| Get Records    | `Get ObjectAPI Purpose` | `Get_ObjectAPI_Purpose` | `Get_Account_Sites`                  |
| Create Records | `Create Purpose`        | `Create_Purpose`        | `Create_SuiteApprovals`              |
| Update Records | `Update Purpose`        | `Update_Purpose`        | `Update_RelatedLcmRelationships`     |
| Subflow        | `Call SubflowAPI`       | `Call_SubflowAPI`       | `Call_SubflowGetSuitesRelatedToSite` |
| Invocable Apex | `Call ApexMethodName`   | `Call_ApexMethodName`   | `Call_GetRankedSuitesForSite`        |
| Loop           | `Loop CollName`         | `Loop_CollName`         | `Loop_DailyCapacityIds`              |
| Decision Node  | `Check Purpose`         | `Check_Purpose`         | `Check_SuiteDemandTypeApprovals`     |

### Flow Best Practices

- Run Apex Tests after any change to a record-triggered flow or its subflows/invocable classes.
- Keep same-record updates in Before Save flows. Split New / Existing / All logic at the flow start.
- Descriptions **MUST** include: flow purpose (append per version, do not overwrite), decision node logic, formula variables.
- Relabel "Default Outcome" on every Decision Node to a meaningful term.
- Do not create cascading decision nodes with no action — consolidate.
- Use Custom Error element for validation messages from flows — do not use checkbox-driven Validation Rules.
- Use subflows for reusable or complex logic. Isolate DMLs in the main flow; pass assignments back from subflows.
- Move complex list/map logic into invocable Apex.
- Add Fault Paths on all Get and DML elements.
- Consolidate DMLs for the same record toward the end of the flow using an accumulating record variable.
- Add null/empty checks before Delete elements. Confirm values changed before any DML.
- Add indentation to formula variables.
- Use Custom Metadata for data-driven logic — avoid hardcoded values or IDs.
- For screen flows requiring system context: isolate only those pieces into system-context subflows. **Do NOT run the entire screen flow in System Context.**
- Use Pause or Screen elements to break up transactions when needed.
- When navigating back in Screen Flows, provide a mechanism to retain previously entered values.

### Flow Absolute Don'ts

- No Get/Create/Update (pink DML) elements inside a loop.
- No flows with no entry criteria defined.
- No hardcoded IDs.
- No querying related records accessible via the trigger record's lookup fields — they are already available on the record variable.

---

## 9. Bypass Mechanism

Create a Hierarchy Custom Setting with the following checkboxes. Every trigger and record-triggered flow must check the relevant flag and exit early when set.

| Field                        | Purpose                                                       |
| ---------------------------- | ------------------------------------------------------------- |
| `Bypass_All_Flows__c`        | Prevents all record-triggered flows from executing            |
| `Bypass_All_Validations__c`  | Prevents all validation rules from executing                  |
| `Bypass_All_Triggers__c`     | Prevents all Apex triggers from executing                     |
| `Bypass_Object_Trigger__c`   | Prevents triggers for objects in `Skipped_Objects_Trigger__c` |
| `Bypass_Object_Flow__c`      | Prevents flows for objects in `Skipped_Objects_Flow__c`       |
| `Skipped_Objects_Trigger__c` | Comma-separated object API names to skip for triggers         |
| `Skipped_Objects_Flow__c`    | Comma-separated object API names to skip for flows            |

---

## 10. OmniStudio Standards

### Component Roles

| Component              | Use Case                                                               |
| ---------------------- | ---------------------------------------------------------------------- |
| OmniScript             | Step-by-step guided user flows, multi-step forms/wizards               |
| DataRaptors            | ETL — extract, transform, load between Salesforce and external systems |
| Integration Procedures | Server-side orchestration, complex API calls, large datasets           |
| FlexCards              | Dynamic user-facing display — dashboards, 360 views, record pages      |

### OmniScript

- Names: PascalCase, prefixed by functional area (e.g., `OrderReservationCreation`)
- Steps and elements: camelCase, named by functionality (e.g., `getOrderInfo`)
- Subflow names: PascalCase with specific function (e.g., `MobilizationNumberCheck`)
- Always start from the OOTB Vlocity Template Designer baseline.
- One clear action per step — keep steps small and modular.
- No complex logic in OmniScripts — delegate to Integration Procedures or Apex.
- Use DataRaptors for transformation/extraction; Integration Procedures for all backend/API logic.
- Use DataRaptor Turbo for high-volume extraction only.
- Never hardcode IDs or codes — use Custom Metadata or Custom Settings.
- Use Error Step for custom error handling.
- Encapsulate reused logic in subflows.

### DataRaptor Naming and Limits

| Type          | Convention                  | Example                               |
| ------------- | --------------------------- | ------------------------------------- |
| Extract       | `DR_Extract_<Feature>`      | `DR_Extract_OrderDetails`             |
| Load          | `DR_Load_<Feature>`         | `DR_Load_SiteInfo`                    |
| Transform     | `DR_Transform_<Feature>`    | `DR_Transform_OrderDetails`           |
| Turbo Extract | `DR_TurboExtract_<Feature>` | `DR_TurboExtract_DailyCapacitiesData` |

**Size limits:**

- Extract: ≤5 queries, minimal formulas, ≤30 output fields
- Transform: ≤200 output fields
- Load: ≤3 objects

- Separate extraction and transformation into distinct DataRaptors.
- Default null values to prevent transformation errors.
- For large datasets, use Turbo Extract with chunked retrieval.
- Implement error logging for all failed operations.

### Integration Procedure Naming

| Type        | Convention                 | Example                              |
| ----------- | -------------------------- | ------------------------------------ |
| Extract     | `IP_Extract_<Feature>`     | `IP_Extract_OrderDetails`            |
| Load        | `IP_Load_<Feature>`        | `IP_Load_SiteInfo`                   |
| Transform   | `IP_Transform_<Feature>`   | `IP_Transform_OrderDetails`          |
| Orchestrate | `IP_Orchestrate_<Feature>` | `IP_Orchestrate_DailyCapacitiesData` |

- Group multiple related API calls into one Action Set.
- Use async processing for long-running operations.
- Focus on orchestrating data flow — not heavy calculations.
- Keep action count minimal to avoid bottlenecks.
- Use variables for temporary data storage.
- Use JSON path expressions for deeply nested API response data.
- Implement error logging for all failed operations.

### FlexCard Naming

- Format: `FC_<ObjectOrArea>_<Feature>` (e.g., `FC_Site_Creation`, `FC_OrderReservation_Reschedule`)
- Name + Author combination must be unique in the org.
- Names: alphanumeric and underscores only; begin with a letter; no spaces; no trailing underscore; no two consecutive underscores.
- Display only — business logic belongs in Integration Procedures, OmniScripts, or Apex.
- Use filters and limits for dynamic data retrieval.
- Query and display only required fields.
- Use conditional visibility based on input, permissions, or values.
- Make FlexCards responsive — use the Viewport dropdown in FlexCard Designer.
- Optimize all images and icons for size and format.

---

## 11. Definition of Done — Every task is complete only when

- [ ] Naming conventions followed for all artifacts.
- [ ] No hardcoded IDs, URLs, usernames, or passwords anywhere.
- [ ] ≥85% Apex code coverage with meaningful assertions.
- [ ] No SOQL or DML inside loops.
- [ ] `with sharing` / `without sharing` declared explicitly on every Apex class with documented rationale.
- [ ] FLS enforced via `Schema` or `Security.stripInaccessible()` for all user-facing data access.
- [ ] Bypass mechanism checked at the start of every trigger and as entry criteria in every record-triggered flow.
- [ ] Flows: Trigger Orders set, Fault Paths on all DML/Get elements, descriptions on nodes and formulas.
- [ ] OmniStudio: naming conventions followed, DataRaptor/IP size limits respected.
- [ ] Test data created within test classes; tests cover positive, negative, bulk, and single-record paths.
- [ ] Latest Salesforce API version on all saved metadata.
- [ ] Code commented for non-obvious logic, sharing decisions, and any bypass justifications.
- [ ] All existing tests pass — no regressions.

## Naming Conventions

| Artifact           | Convention                              |
| ------------------ | --------------------------------------- |
| Trigger            | `<SObject>Trigger`                      |
| Trigger Handler    | `<SObject>TriggerHandler`               |
| Service            | `<Functionality>Service`                |
| Domain             | `<Functionality>Domain`                 |
| Selector           | `<SObject>Selector`                     |
| Batch              | `<Functionality>Batch`                  |
| Schedulable        | `<Functionality>Schedulable`            |
| Queueable          | `<Functionality>Queueable`              |
| Utility            | `<Functionality>Utility`                |
| Test class         | `<ClassName>Test`                       |
| LWC component      | camelCase folder → kebab-case tag       |
| Flow (Before Save) | `ObjectAPI-BeforeSave-Process`          |
| Flow (After Save)  | `ObjectAPI-AfterSave-Process`           |
| Flow (Screen)      | `Screenflow-Feature`                    |
| Flow (Scheduled)   | `Scheduled-ObjectAPI-Feature`           |
| Permission Set     | `<Object Name> Permissions`             |
| Sharing Rule       | `Share[Object][Recipient]`              |
| Field API name     | PascalCase, no underscores except `__c` |
