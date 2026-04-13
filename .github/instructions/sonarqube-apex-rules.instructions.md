---
applyTo: "**/*.cls,**/triggers/**,**/*.trigger"
description: "SonarQube and PMD static analysis rules for Apex. Run this checklist against every Apex class and trigger before deployment. Covers Best Practices, Code Style, Design, Error Prone, Performance, and Security rule categories. Flag any violation as a blocker — do not deploy code that fails a Critical or High rule."
---

# SonarQube / PMD — Apex Static Analysis Rules

Run every rule in this file against all `.cls` and `.trigger` files before deploying to any org. Violations marked **🔴 Critical** or **🟠 High** are deployment blockers. Fix them before proceeding.

---

## How to Run

```bash
# PMD via Salesforce CLI plugin
sf scanner run --target "force-app/main/default/classes,force-app/main/default/triggers" \
  --engine pmd \
  --category "Best Practices,Code Style,Design,Error Prone,Performance,Security" \
  --format table

# Or scan a single file
sf scanner run --target "force-app/main/default/classes/MyClass.cls" --format table
```

Reference: [Salesforce Code Analyzer](https://forcedotcom.github.io/sfdx-scanner/) | [PMD Apex Rules](https://pmd.github.io/pmd/pmd_rules_apex.html)

---

## Category 1 — Best Practices 🔴

### ApexAssertionsShouldIncludeMessage — 🔴 Critical

Every `System.assert()` call must include a descriptive failure message.

```apex
// ❌ Fails rule
System.assert(result != null);
System.assertEquals(expected, actual);

// ✅ Passes rule
System.assert(result != null, 'Result should not be null after service call');
System.assertEquals(expected, actual, 'Account name mismatch after update');
```

### ApexUnitTestClassShouldHaveAsserts — 🔴 Critical

Every `@isTest` class must contain at least one `System.assert`, `assertEquals`, or `assertNotEquals`.

- Tests with no assertions prove nothing and give false confidence in coverage.

### ApexUnitTestClassShouldHaveRunAs — 🟠 High

Every test class must contain at least one `System.runAs()` call.

- Tests must verify behavior under a specific user context, not just system context.

### ApexUnitTestMethodShouldHaveIsTestAnnotation — 🟡 Medium

Use `@isTest` annotation on test methods — never use the deprecated `testMethod` keyword.

```apex
// ❌ Deprecated
static testMethod void myTest() {}

// ✅ Correct
@isTest
static void myTest() {}
```

### ApexUnitTestShouldNotUseSeeAllDataTrue — 🔴 Critical

Never use `@isTest(seeAllData=true)`. Tests must create their own data.

```apex
// ❌ Deployment blocker
@isTest(seeAllData=true)
class MyTest {}

// ✅ Correct
@isTest
class MyTest {}
```

### AvoidFutureAnnotation — 🟠 High

Prefer `Queueable` over `@future`. Future methods cannot be chained, have no Job ID, and accept only primitive parameters.

- Exception: callouts where `@future(callout=true)` is the only option and Queueable is not feasible.

### AvoidGlobalModifier — 🟠 High

Do not declare classes or methods as `global` unless required by a managed package or interface contract. Global artifacts can never be deleted from an org.

### AvoidLogicInTrigger — 🔴 Critical

Trigger bodies must contain no business logic. All logic must be delegated to a `TriggerHandler` class.

```apex
// ❌ Deployment blocker
trigger AccountTrigger on Account (before insert) {
    for (Account a : Trigger.new) {
        a.Name = a.Name.toUpperCase(); // logic in trigger
    }
}

// ✅ Correct
trigger AccountTrigger on Account (before insert) {
    AccountTriggerHandler.getInstance().beforeInsert(Trigger.new);
}
```

### DebugsShouldUseLoggingLevel — 🟡 Medium

Always specify a `LoggingLevel` as the first parameter when using `System.debug` with two parameters.

```apex
// ❌ Fails rule
System.debug('Processing record: ' + recordId);

// ✅ Correct
System.debug(LoggingLevel.DEBUG, 'Processing record: ' + recordId);
```

### QueueableWithoutFinalizer — 🟠 High

Any class implementing `Queueable` must attach a `Finalizer` to handle success and failure states.

```apex
public class MyQueueable implements Queueable {
    public void execute(QueueableContext ctx) {
        System.attachFinalizer(new MyFinalizer()); // required
        // job logic
    }
}
```

### UnusedLocalVariable — 🟡 Medium

Remove all declared local variables that are never referenced. Dead variables indicate incomplete refactoring.

---

## Category 2 — Code Style 🟡

### AnnotationsNamingConventions — 🟡 Medium

Apex annotations must use their canonical casing: `@isTest`, `@AuraEnabled`, `@InvocableMethod`, `@future`, `@TestSetup`, etc.

### ClassNamingConventions — 🟠 High

Class names must follow PascalCase. Match project naming conventions (see Project Guidelines):

- `AccountTriggerHandler` ✅
- `accounttriggerhandler` ❌

### FieldDeclarationsShouldBeAtStart — 🟡 Medium

All field/property declarations must appear before any method declarations within a class.

### FieldNamingConventions — 🟠 High

Instance fields must follow Hungarian notation prefix as defined in Project Guidelines (e.g., `bIsActive`, `sAccountName`, `lstContacts`).

### ForLoopsMustUseBraces — 🟡 Medium

All `for` loops must use `{}` braces even for single-line bodies.

```apex
// ❌
for (Account a : accounts) a.Name = 'Test';

// ✅
for (Account a : accounts) {
    a.Name = 'Test';
}
```

### FormalParameterNamingConventions — 🟡 Medium

Method parameter names must be camelCase and follow Hungarian notation.

### IfElseStmtsMustUseBraces / IfStmtsMustUseBraces — 🟡 Medium

All `if` and `if/else` blocks must use `{}` braces.

### LocalVariableNamingConventions — 🟡 Medium

Local variables must follow camelCase and Hungarian notation prefix.

### MethodNamingConventions — 🟠 High

Method names must be camelCase, start with a verb, and reflect the return type where meaningful. Example: `getAccountsByOwnerId`.

### OneDeclarationPerLine — 🟡 Medium

Declare only one variable per line.

```apex
// ❌
String sFirst, sLast;

// ✅
String sFirst;
String sLast;
```

### PropertyNamingConventions — 🟡 Medium

Properties must follow camelCase naming.

### WhileLoopsMustUseBraces — 🟡 Medium

All `while` loops must use `{}` braces.

---

## Category 3 — Design 🟠

### AvoidBooleanMethodParameters — 🟡 Medium

Avoid Boolean parameters in public APIs — they make call sites unreadable. Use an enum or separate methods instead.

```apex
// ❌ Unclear at call site: processOrder(true, false)
public void processOrder(Boolean bSendEmail, Boolean bCreateTask) {}

// ✅
public void processOrderWithEmail() {}
public void processOrderSilently() {}
```

### AvoidDeeplyNestedIfStmts — 🟠 High

Maximum nesting depth for `if` statements: **3 levels**. Extract nested logic into private methods or use early returns.

### CognitiveComplexity — 🟠 High

Cognitive complexity score must not exceed **15 per method**. Refactor complex methods into smaller units.

### CyclomaticComplexity — 🟠 High

Cyclomatic complexity must not exceed **10 per method**. Each decision point (if, for, while, case, catch) adds 1.

### ExcessiveParameterList — 🟠 High

Methods must not have more than **4 parameters**. Use a wrapper/DTO class for methods requiring more inputs.

### ExcessivePublicCount — 🟡 Medium

Classes must not expose more than **20 public methods/properties**. Large public APIs signal a class doing too much.

### NcssCount — 🟡 Medium

Non-Commenting Source Statements (NCSS) per method: max **60**. Per class: max **500**.

### TooManyFields — 🟡 Medium

Classes must not declare more than **15 fields**. Excessive fields indicate the class needs decomposing.

### UnusedMethod — 🟠 High

Remove all private methods that are never called. Dead code increases maintenance burden and confuses reviewers.

---

## Category 4 — Documentation 🟡

### ApexDoc — 🟡 Medium

All `public` and `global` classes, methods, and properties must have ApexDoc comments:

```apex
/**
 * @description Retrieves accounts filtered by owner IDs.
 * @param ownerIds Set of owner record IDs to filter by.
 * @return List of matching Account records.
 */
public static List<Account> byOwnerIds(Set<Id> ownerIds) {}
```

---

## Category 5 — Error Prone 🔴

### ApexCSRF — 🔴 Critical

Never perform DML in Apex class constructors or static initializers. This causes unexpected side effects and CSRF vulnerabilities.

### AvoidDirectAccessTriggerMap — 🟠 High

Never access `Trigger.old` or `Trigger.new` directly by index without null/size checks. Always pass collections to handler methods.

```apex
// ❌ Index access without guard
Account a = Trigger.new[0];

// ✅ Pass full list to handler
handler.beforeInsert(Trigger.new);
```

### AvoidHardcodingId — 🔴 Critical

Never hardcode Salesforce record IDs, Profile IDs, RecordType IDs, or any org-specific IDs. Use Custom Metadata, Custom Settings, or SOQL lookups.

```apex
// ❌ Deployment blocker
Id adminProfileId = '00e000000000001';

// ✅
Id adminProfileId = [SELECT Id FROM Profile WHERE Name = 'System Administrator' LIMIT 1].Id;
```

### AvoidNonExistentAnnotations — 🟡 Medium

Do not use unsupported or misspelled annotations. Only use platform-supported annotations.

### AvoidStatefulDatabaseResult — 🟠 High

Do not store `Database.SaveResult`, `Database.UpsertResult`, or similar DML result types as instance variables in `Database.Stateful` batch classes. These types are not serializable.

### EmptyCatchBlock — 🔴 Critical

Never leave a `catch` block empty. At minimum, log the exception.

```apex
// ❌ Deployment blocker
try {
    update records;
} catch (Exception e) {
    // silent failure
}

// ✅
try {
    update records;
} catch (Exception e) {
    Logger.error('Update failed', e);
}
```

### EmptyIfStmt / EmptyStatementBlock / EmptyTryOrFinallyBlock / EmptyWhileStmt — 🟠 High

Remove all empty `if`, `while`, `try`, and `finally` blocks. They serve no purpose and indicate incomplete implementation.

### InaccessibleAuraEnabledGetter — 🔴 Critical

Properties annotated with `@AuraEnabled` must have `public` or `global` access modifiers on their getter. Private getters on AuraEnabled properties cause runtime errors since Summer '21.

### MethodWithSameNameAsEnclosingClass — 🟠 High

Non-constructor methods must not share the name of their enclosing class. This causes ambiguity and compilation warnings.

### OverrideBothEqualsAndHashcode — 🟠 High

If you override `equals()`, you must also override `hashCode()`, and vice versa. Inconsistent implementations break collections.

### TestMethodsMustBeInTestClasses — 🔴 Critical

Methods annotated with `@isTest` or `testMethod` must only appear in `@isTest` classes.

### TypeShadowsBuiltInNamespace — 🟠 High

Do not name Apex classes, enums, or interfaces with the same name as a built-in Salesforce type (`Database`, `System`, `Schema`, `Math`, etc.).

---

## Category 6 — Performance 🔴

### AvoidDebugStatements — 🟠 High

Remove or guard `System.debug` calls in production code. Debug statements consume Apex CPU time even when debug logs are disabled.

```apex
// ❌ In production code
System.debug('Account processed: ' + accountId);

// ✅ Guard with a custom switch or remove entirely
if (Logger.isDebugEnabled()) {
    System.debug(LoggingLevel.DEBUG, 'Account processed: ' + accountId);
}
```

### AvoidNonRestrictiveQueries — 🔴 Critical

All SOQL queries must have a `WHERE` clause with at least one indexed, selective filter. Queries without filters on large objects cause full-table scans and governor limit violations.

```apex
// ❌ Deployment blocker
List<Account> accounts = [SELECT Id FROM Account];

// ✅
List<Account> accounts = [SELECT Id FROM Account WHERE OwnerId IN :ownerIds LIMIT 200];
```

### EagerlyLoadedDescribeSObjectResult — 🟡 Medium

Use `SObjectType.getSObjectType().getDescribe(SObjectDescribeOptions.DEFERRED)` for describe calls that don't need all metadata immediately. Eager describes slow transaction start time.

### OperationWithHighCostInLoop — 🔴 Critical

Never call the following inside loops:

- `String.format()`
- `Json.serialize()` / `Json.deserialize()`
- `Type.forName()`
- `Schema.describeSObjects()`

### OperationWithLimitsInLoop — 🔴 Critical

**Deployment blocker.** Never place inside any loop:

- SOQL queries (`[SELECT ...]`)
- SOSL queries (`[FIND ...]`)
- DML statements (`insert`, `update`, `delete`, `upsert`, `merge`)
- `Database.*` methods
- `Approval.process()`
- `Messaging.sendEmail()`

Collect all records into a List/Map first, then perform a single bulk operation outside the loop.

---

## Category 7 — Security 🔴

### ApexBadCrypto — 🔴 Critical

Never hardcode initialization vectors (IVs) or encryption keys in `Crypto` API calls. Always use `Crypto.generateAesKey()` or `Crypto.getRandomInteger()` to generate keys/IVs at runtime.

### ApexCRUDViolation — 🔴 Critical

Check CRUD permissions before every SOQL/DML operation in user-facing Apex:

```apex
// Before INSERT
if (!Schema.sObjectType.Account.isCreateable()) {
    throw new AuraHandledException('Insufficient permissions to create Account');
}

// Before QUERY
if (!Schema.sObjectType.Account.isAccessible()) { ... }

// Before UPDATE
if (!Schema.sObjectType.Account.isUpdateable()) { ... }

// Before DELETE
if (!Schema.sObjectType.Account.isDeletable()) { ... }
```

Or use `Security.stripInaccessible(AccessType.READABLE, records)` for field-level enforcement.

### ApexDangerousMethods — 🔴 Critical

The following method calls are flagged as dangerous and must not be used:

- `FinancialForce.Security.bypass()` — bypasses all security
- Any method that grants elevated permissions without explicit justification

### ApexInsecureEndpoint — 🔴 Critical

Never call HTTP endpoints over plain `http://`. All callouts must use `https://`.

```apex
// ❌ Deployment blocker
Http h = new Http();
HttpRequest req = new HttpRequest();
req.setEndpoint('http://api.example.com/data');

// ✅
req.setEndpoint('https://api.example.com/data');
// Better: use Named Credential
req.setEndpoint('callout:MyNamedCredential/data');
```

### ApexOpenRedirect — 🔴 Critical

Never redirect to a URL built from user-controlled input without validation. Validate and whitelist redirect targets.

### ApexSharingViolations — 🔴 Critical

Every Apex class that performs DML or SOQL must explicitly declare `with sharing` or `without sharing`. Classes without an explicit declaration are a deployment blocker.

```apex
// ❌ Deployment blocker
public class AccountService {
    public void updateAccounts(List<Account> accounts) {
        update accounts;
    }
}

// ✅
public with sharing class AccountService {
    public void updateAccounts(List<Account> accounts) {
        update accounts;
    }
}
```

### ApexSOQLInjection — 🔴 Critical

Never concatenate unescaped user input into dynamic SOQL. Always use `String.escapeSingleQuotes()` or bind variables.

```apex
// ❌ Deployment blocker — SQL injection risk
String query = 'SELECT Id FROM Account WHERE Name = \'' + userInput + '\'';

// ✅
String query = 'SELECT Id FROM Account WHERE Name = \''
    + String.escapeSingleQuotes(userInput) + '\'';

// ✅ Best — use bind variable
List<Account> accounts = [SELECT Id FROM Account WHERE Name = :userInput];
```

### ApexSuggestUsingNamedCred — 🔴 Critical

Never hardcode credentials, tokens, usernames, or passwords in Apex. Always use Named Credentials or Protected Custom Metadata.

```apex
// ❌ Deployment blocker
req.setHeader('Authorization', 'Bearer eyJhbGciOiJSUzI1NiIs...');

// ✅
req.setEndpoint('callout:MyNamedCredential/endpoint');
```

### ApexXSSFromEscapeFalse — 🔴 Critical

Never call `addError(message, false)` with escaping disabled. The unescaped message renders raw HTML and opens XSS attack surface.

```apex
// ❌ Deployment blocker
record.addError('<script>alert(1)</script>', false);

// ✅ — escaping is enabled by default
record.addError('Validation failed: invalid status value.');
```

### ApexXSSFromURLParam — 🔴 Critical

All values read from URL parameters in Visualforce or Apex Pages controllers must be escaped before rendering. Use `HTMLENCODE()` in Visualforce or `String.escapeSingleQuotes()` in Apex.

---

## Pre-Deployment Checklist

Run this before every `sf project deploy` or PR merge:

- [ ] `sf scanner run` exits with zero Critical/High violations
- [ ] No SOQL or DML inside any loop
- [ ] No hardcoded IDs, credentials, or `http://` endpoints
- [ ] All classes declare `with sharing` or `without sharing` with a justification comment
- [ ] No empty catch blocks
- [ ] All test classes have at least one assertion and one `runAs`
- [ ] No `@isTest(seeAllData=true)`
- [ ] No logic in trigger bodies
- [ ] Cyclomatic complexity ≤10 per method
- [ ] All `@AuraEnabled` property getters are `public` or `global`
- [ ] No `System.debug` statements left in production paths without a guard

---

## References

- [PMD Apex Rules Index](https://pmd.github.io/pmd/pmd_rules_apex.html)
- [Salesforce Code Analyzer (sf scanner)](https://forcedotcom.github.io/sfdx-scanner/)
- [Enforcing CRUD and FLS](https://developer.salesforce.com/page/Enforcing_CRUD_and_FLS)
- [Apex Security Best Practices](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing_chapter.htm)
- [PMD Quickstart Ruleset](https://github.com/pmd/pmd/blob/master/pmd-apex/src/main/resources/rulesets/apex/quickstart.xml)
- [Apex Governor Limits](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_apexgov.htm)
