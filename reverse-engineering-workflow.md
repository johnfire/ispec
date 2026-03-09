# Reverse Engineering Workflow
## From Existing Code to .ispec
*Use this document to instruct Claude Code to generate an .ispec file from an existing codebase.*

---

## When To Use This

- You have an existing project and want to bring it into the .ispec system
- A client has handed you a codebase you need to understand quickly
- You want to generate living documentation for code that has none
- You want to validate the .ispec compiler by round-tripping: code → .ispec → code

---

## What You Need Before Starting

- The codebase accessible to Claude Code
- The ispec-language-spec.md in the same working directory
- A clear idea of which domain or module you want to reverse engineer
  (do one domain at a time — do not attempt the whole codebase at once)

---

## The Prompt Template

Copy this prompt into Claude Code. Fill in the bracketed sections.

```
Read ispec-language-spec.md for the .ispec language rules.
You are going to reverse engineer existing code into an .ispec file.

TARGET CODE: [path to the file or folder to reverse engineer]
DOMAIN NAME: [what to call this domain, e.g. "UserAccounts", "Orders"]
OUTPUT FILE: [where to write the .ispec, e.g. specs/user-accounts.ispec]

Follow these steps exactly. Do one step at a time and stop for review.

STEP 1 — READ THE CODE
Read all files in the target. Do not generate anything yet.
Identify and list:
  - All data models, classes, schemas, or database tables
  - All functions, methods, or API routes that change state
  - All functions, methods, or API routes that read state
  - All error types, exception classes, or error codes
  - Any event emission or message publishing
  - Any explicit business rules or validation logic

Present this list. Stop and wait for approval before continuing.

STEP 2 — IDENTIFY ENTITIES
From the data models found in Step 1:
  - Name each entity
  - List its fields and types
  - Identify which fields are: unique, generated, immutable, optional
  - Identify the history strategy: does this data need full history (immutable)
    or is it overwritten in place (transient)?
  - Identify relationships between entities

Present the entity list in plain English. Stop and wait for approval.

STEP 3 — IDENTIFY OPERATIONS
From the state-changing functions found in Step 1:
  - Name each operation (use the .ispec naming convention: verb + noun)
  - Identify inputs
  - Extract preconditions from: guard clauses, if-statements, validation checks
  - Extract postconditions from: what the function guarantees on success
  - Extract failure modes from: exceptions thrown, error returns, error codes
  - Identify side effects: events emitted, emails sent, external calls made
  - Note anything that looks like a business rule buried in the logic

Present operations in plain English. Flag anything unclear or ambiguous.
Stop and wait for approval.

STEP 4 — IDENTIFY QUERIES
From the state-reading functions found in Step 1:
  - Name each query
  - Identify what entity is returned
  - Extract filter conditions
  - Extract sort order if present
  - Note whether it returns one or many results
  - Identify any joins or relationship traversals

Present queries in plain English. Stop and wait for approval.

STEP 5 — IDENTIFY EVENTS
From any event emission, message publishing, or webhook calls:
  - Name each event
  - List the data it carries
  - Identify which domain owns it

If no events are found, note that explicitly.
Stop and wait for approval.

STEP 6 — FLAG AMBIGUITIES
Before generating the .ispec, list everything that is unclear:
  - Business rules that appear to exist but whose intent is not obvious
  - Error handling that looks wrong or incomplete
  - Operations that do too many things (should be split)
  - Data that is mutated in ways that suggest history is being lost
  - Anything that looks like a bug rather than a feature
  - Anything the code does that contradicts what comments say it does

This step is important. The gap between what the code does and what it
was supposed to do is where bugs live. Surface it now.
Stop and wait for review and clarification.

STEP 7 — GENERATE THE .ispec
Using the approved output from Steps 2-6, generate the complete .ispec file.
Follow the ispec-language-spec.md rules exactly.
Generate both the [human] section and the [spec] section.
Generate a [policy] section for any domain-specific rules found.

In the [human] section:
  - Describe entities in plain English
  - Describe operations in plain English
  - Describe queries in plain English
  - Use the language a non-technical client could read and verify
  - Flag anything that was ambiguous in Step 6 with a note:
    "NOTE: This behavior was inferred from the code. Verify intent."

In the [spec] section:
  - Generate all entities with correct types and constraints
  - Generate all operations with requires, ensures, failures
  - Generate all queries with correct syntax
  - Generate all events
  - Generate @extend index hints based on queries found
  - Generate @extend computed for any derived fields found

Stop and present the complete .ispec. Wait for review.
```

---

## After The .ispec Is Generated

### Review Checklist

Go through the generated .ispec carefully. For each section, ask:

**Entities:**
- [ ] Does every entity match what the code actually stores?
- [ ] Are the field types correct domain types (not primitives)?
- [ ] Is the history setting right? (immutable vs transient)
- [ ] Are relationships between entities correctly represented?

**Operations:**
- [ ] Does each operation name describe what it does clearly?
- [ ] Do the requires clauses capture all the real preconditions?
- [ ] Do the ensures clauses describe what the code actually guarantees?
- [ ] Are all real failure modes listed?
- [ ] Are there failures in the code that are missing from the spec?
- [ ] Are there things the spec says should fail that the code actually allows?

**Queries:**
- [ ] Does each query return what the code actually returns?
- [ ] Are filters correct?
- [ ] Is sort order correct?

**The Important Question:**
- [ ] Is there a difference between what the code DOES and what it was SUPPOSED to do?
  - If yes: fix the code, not the spec. The spec should reflect intent, not bugs.

---

## Common Patterns Found In Existing Code

### God Functions
A function that does too many things. Common in older code.

```
# code found:
def process_order(order, payment, user):
    validate_order(order)
    charge_payment(payment)
    update_inventory(order)
    send_confirmation(user)
    log_analytics(order)
```

**What to do:** Split into separate operations in the .ispec.
`ValidateOrder`, `ChargePayment`, `UpdateInventory` — each with their
own requires, ensures, and failures. Connect them with events.

---

### Silent Failures
Code that swallows exceptions or returns None on failure.

```
# code found:
def get_user(email):
    try:
        return User.query.filter_by(email=email).first()
    except:
        return None
```

**What to do:** In the .ispec, make the failure explicit.
```
@query GetUserByEmail
  find:    User
  where:   User.email == email
  returns: one

failures:
  UserNotFound: NotFoundError
    condition: no User matches email
```

---

### Implicit Business Rules
Logic buried in code with no explanation.

```
# code found:
if order.total > 10000 and not user.verified:
    raise Exception("limit exceeded")
```

**What to do:** Surface it as an explicit policy and failure.
```
[policy]
  OrderPolicy:
    unverified user maximum order value: 10000

failures:
  UnverifiedUserLimitExceeded: PolicyViolation
    condition: order.total > OrderPolicy.unverified_limit
               and user.verified == false
```

---

### Missing History
Code that overwrites data that should be preserved.

```
# code found:
user.email = new_email
db.session.commit()
```

**What to do:** Flag this during Step 6. In the .ispec, mark the entity
as `history: immutable`. The compiler will generate append-only storage.
The old email will be preserved.

---

### Orphaned Error Types
Exception classes defined but not consistently raised.

**What to do:** During Step 6, flag these. Either they belong in the
failure hierarchy and should be used, or they are dead code and should
be removed. The .ispec forces a decision.

---

## The Round-Trip Test

Once you have the .ispec, you can validate both the spec and the compiler:

```
1. Generate .ispec from existing code   (this workflow)
2. Review and approve the .ispec
3. Run the .ispec through the compiler  (forward pipeline)
4. Compare compiler output to original code

Differences fall into three categories:

  a) Compiler is wrong
     The spec was correct but the compiler generated
     different code. Fix the compiler.

  b) Original code was wrong
     The spec revealed that the original code had a bug
     or missing behavior. Fix the original code.

  c) Spec language gap
     The spec could not express something the original
     code did. Add the construct to the spec language.
```

This round-trip is the fastest way to find gaps in both
the compiler and the spec language.

---

## Output File Naming

```
specs/
  user-accounts.ispec
  orders.ispec
  payments.ispec
  client-tracker.ispec
```

One domain per file. Name the file after the domain in kebab-case.
Match the domain name in the [spec] section.

---

*Reverse Engineering Workflow — Version 0.1*
*Part of the .ispec language system*
