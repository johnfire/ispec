# Reverse .ispec System
## From existing codebase to complete framework documents

*This workflow takes an existing codebase and produces a complete set of
.ispec framework documents — one .ispec file per domain, plus arch.md
and context.md. It works in two phases: discovery first, generation second.
Do not skip the discovery phase.*

---

## What This Workflow Produces

```
INPUT:   existing codebase (any language, any age)

OUTPUT:
  context.md              ← what the system does and why
  arch.md                 ← how it is structured and why
  specs/
    domain-one.ispec      ← one per identified domain
    domain-two.ispec
    ...
  ispec system/
    global.policy         ← error hierarchy and global rules
    .intent               ← constitutional layer (draft)
```

---

## Honest Scope Statement

**This workflow handles well:**
- Modern, reasonably structured codebases
- Python, JavaScript, TypeScript, Kotlin, Java, Go
- Projects with clear module or package boundaries
- REST APIs, web applications, backend services
- Codebases with some existing documentation

**This workflow handles with more effort:**
- Legacy codebases (PHP, older Java, COBOL, PL/1)
- Monoliths with no clear module boundaries
- Codebases with significant technical debt
- Systems with undocumented business rules buried in logic
- Code that mixes concerns extensively

**For legacy and heavily tangled codebases:**
The workflow still applies but expect more `[UNKNOWN]` and
`[INFERRED]` markers in the output. More human input is required.
More review cycles are needed. The discovery phase becomes more
important, not less. Do not rush it.

**This workflow cannot recover:**
- Intent that was never written down anywhere
- Business rules that exist only in someone's head
- The reasons behind architectural decisions
- Why something was built the way it was

These gaps will be marked explicitly. They require human input.

---

## Before You Start

**What you need:**
- The codebase accessible to Claude Code
- `ispec system/ispec-language-spec.md` in your working directory
- Enough time — this is not a five-minute task for a real codebase
- A human who knows the system available for the review stages

**Set expectations:**
The output of this workflow is a draft, not a finished product.
The discovery report requires human review before generation begins.
The generated documents require human review and correction.
Context.md will have gaps that only a human can fill.
Arch.md will be missing the reasons behind decisions.

The workflow produces the structure. Humans provide the knowledge
that was never in the code.

---

## Phase 1 — Discovery

**Goal:** Understand the full system before generating anything.
**Output:** A discovery report the human reviews and approves.
**Rule:** Do not generate any .ispec, arch.md, or context.md during Phase 1.

### Phase 1 Prompt

Copy this into Claude Code:

```
Read "ispec system/ispec-language-spec.md" for the .ispec language rules.

You are going to reverse engineer an existing codebase into .ispec
framework documents. This is Phase 1 — Discovery. Do not generate
any .ispec files, arch.md, or context.md during this phase.
Your only output in this phase is a discovery report.

TARGET CODEBASE: [path to codebase root]

Follow these steps exactly. Complete all steps before writing
the discovery report.

STEP 1 — READ THE STRUCTURE
Read the directory structure of the entire codebase.
Do not read file contents yet — read structure only.
Identify:
  - Top-level organization (modules, packages, folders)
  - Configuration files and what they reveal about the stack
  - Test files and what they reveal about the architecture
  - Documentation files (README, docs/, wiki/)
  - Database migration files or schema definitions
  - API definition files (OpenAPI, GraphQL schema, protobuf)

STEP 2 — READ EXISTING DOCUMENTATION
Read all documentation files before reading any code.
Documentation reveals intent that code cannot.
Read:
  - README files at all levels
  - Any docs/ or documentation/ folders
  - Inline module-level documentation
  - API documentation if present
  - Database schema comments
  - Any architecture decision records (ADRs)

STEP 3 — READ CONFIGURATION AND SCHEMA
Read configuration files, schema definitions, and migrations.
These reveal the data model more reliably than application code.
Read:
  - Database schemas and migrations
  - ORM model definitions
  - API schema definitions
  - Environment configuration (structure, not values)
  - Dependency files (requirements.txt, package.json, pom.xml)

STEP 4 — READ APPLICATION CODE
Now read the application code, starting with entry points.
  - Main application files and routing
  - Model / entity definitions
  - Service / business logic layers
  - Data access layers
  - Event handlers and message consumers
  - External integrations

Read in this order: models first, then services, then routes/handlers.
Flag anything that is unclear, appears to be a bug, or contradicts
what the documentation says.

STEP 5 — READ TEST CODE
Test code often reveals intent that application code obscures.
Read all test files. Note:
  - What behaviors are explicitly tested
  - What edge cases are covered
  - What is conspicuously not tested
  - Test names that reveal business rules

STEP 6 — COMPILE THE DISCOVERY REPORT
Write the discovery report in this exact format.
Be specific. Use actual names from the code.
Mark every inference clearly as [INFERRED].
Mark every gap clearly as [UNKNOWN].

---

# Discovery Report
## [Project Name]

### System Summary
One paragraph describing what this system appears to do,
who it appears to serve, and its approximate scale.
Mark inferences.

### Technology Stack
List every technology identified with confidence level:
  Confirmed: explicitly in configuration files
  Inferred:  appears in code but not configuration
  Unknown:   not determinable from code alone

### Identified Domains
For each domain identified:

  Domain: [Name]
  Confidence: High / Medium / Low
  Evidence: [what in the code indicates this is a distinct domain]
  Primary files: [list the key files]
  Entities found: [list]
  Operations found: [list]
  Queries found: [list]
  Events found: [list, if any]
  Dependencies: [other domains this depends on]
  Concerns: [anything unclear, inconsistent, or suspicious]

### Cross-Domain Dependencies
Map of which domains depend on which other domains.
Identify any circular dependencies.
Identify any god classes or modules that everything depends on.

### Unclassified Code
Files or modules that do not fit cleanly into any domain.
These may indicate:
  - Missing domains not yet identified
  - Utility code that belongs in a shared layer
  - Technical debt with no clear home
  - Dead code

### Data Model Overview
Summary of the data model as found in code.
Note any inconsistencies between schema definitions
and application code usage.
Note any fields that appear to be vestigial or unused.

### Security Observations
Note any obvious security concerns found during reading:
  - Authorization patterns (or absence of them)
  - Sensitive data handling
  - Input validation patterns
  - Known vulnerability patterns observed

### Bugs and Anomalies
List anything that appears to be a bug rather than a feature.
List code that contradicts documentation.
List error handling that appears incomplete or incorrect.

### Questions Requiring Human Input
Number each question. These must be answered before
Phase 2 begins.

  1. [Specific question about intent or behavior that
     cannot be determined from the code alone]
  2. ...

### Recommended Domain Boundaries
Based on the analysis, recommended .ispec file structure:

  specs/
    [domain-name].ispec    [rationale]
    [domain-name].ispec    [rationale]

Note any domain boundary decisions that are unclear
and explain the alternatives.

### Confidence Assessment
Overall confidence in the analysis:
  High:   codebase is clean, documented, modern
  Medium: some ambiguity, some gaps, manageable
  Low:    significant complexity, poor documentation,
          heavy human input required

---

Present the discovery report. Stop. Wait for human review
and answers to all questions before proceeding to Phase 2.
```

---

## Phase 1 Review

**Before approving the discovery report, verify:**

```
□ The system summary accurately describes what you know
  about the system

□ The technology stack is correct

□ The domain boundaries make sense
  - Are there domains missing?
  - Are any two domains actually one domain?
  - Are any domains actually two domains that were merged?

□ The entity list for each domain is complete

□ The operations list captures the real business behavior
  - Not just CRUD — the actual business operations

□ The bugs and anomalies list is taken seriously
  - These are not things to fix the spec around
  - They are things to fix in the code

□ All questions have been answered

□ The recommended domain structure is agreed
```

**Do not approve until all questions are answered.**
The questions exist because the AI found gaps it cannot fill
from code alone. Skipping them produces specs that document
what the code does, not what it was supposed to do.

---

## Phase 2 — Generation

**Goal:** Generate all framework documents from the approved discovery report.
**Rule:** One domain at a time. Complete and approve each domain's .ispec
before moving to the next. Do not attempt all domains at once.

### Phase 2a — Generate global.policy

First. Before any .ispec files. The error hierarchy must exist
before operations can reference it.

```
Read "ispec system/ispec-language-spec.md" for the language rules.
Read the approved discovery report.

Generate global.policy for this system.

Base the error hierarchy on:
  - Error types found in the codebase
  - Exception classes defined
  - HTTP status codes used
  - Error handling patterns observed

Follow the global.policy format from the ispec-language-spec.md.
Add any system-specific policies found during discovery
(rate limiting, pagination defaults, retry behavior).

Mark anything inferred from the code as [INFERRED: ...].
Mark anything not determinable from the code as
[UNKNOWN — requires human input].

Output to: ispec system/global.policy
```

---

### Phase 2b — Generate .ispec files (one domain at a time)

Use this prompt for each domain. Replace [DOMAIN NAME] and
[PRIMARY FILES] with values from the discovery report.

```
Read "ispec system/ispec-language-spec.md" for the language rules.
Read "ispec system/global.policy" for the error hierarchy.
Read the approved discovery report.
Read these files for the [DOMAIN NAME] domain:
  [PRIMARY FILES listed in discovery report]

Generate the complete .ispec file for the [DOMAIN NAME] domain.

Follow this process:

STEP 1 — ENTITIES
For each entity identified in the discovery report:
  - Map fields to .ispec domain types (not primitives)
  - Determine history: immutable or history: transient
    for each entity based on the data model
  - Identify relationships using → and →[] syntax
  - Apply appropriate constraints

STEP 2 — ENUMS AND TYPES
Identify all status fields, type fields, and constrained
text fields. Define them as enum or type constructs.

STEP 3 — OPERATIONS
For each operation identified in the discovery report:
  - Map to .ispec operation syntax
  - Extract requires from: guard clauses, if-statements,
    permission checks, validation logic
  - Extract ensures from: what the function guarantees
    on success — what state changes, what is created
  - Extract failures from: exceptions thrown, error returns
  - Identify side effects: events, emails, external calls
  - Determine if idempotent: true applies

STEP 4 — QUERIES
For each query identified:
  - Map to @query syntax
  - Identify find, through (if join), where, order
  - Mark returns: one where appropriate

STEP 5 — EVENTS
For any events, messages, or webhooks identified:
  - Define event constructs
  - Define on Event subscriptions if handlers found

STEP 6 — EXTENSIONS
Generate @extend index hints based on query patterns found.
Generate @extend computed for any calculated fields found.

STEP 7 — GENERATE BOTH SECTIONS
Generate the complete [human] section in plain English.
Generate the complete [spec] section in formal syntax.
Generate the [policy] section for domain-specific rules.

For anything inferred from code: mark as
[INFERRED: describe what was assumed]

For anything that cannot be determined: mark as
[UNKNOWN — requires human input: describe the gap]

For anything that appears to be a bug in the original code:
mark as [POSSIBLE BUG: describe the concern]

Output to: specs/[domain-name].ispec

Stop. Present the generated file. Wait for review and approval
before generating the next domain.
```

**Domain review checklist:**

```
h doc review (non-technical reviewer):
  □ Does this describe the system accurately in plain English?
  □ Are there business rules missing?
  □ Are the failure descriptions correct?
  □ Are [INFERRED] items correct or do they need changing?
  □ Are [UNKNOWN] items now answerable?

s doc review (technical reviewer):
  □ Are entity fields correctly typed?
  □ Are requires conditions complete?
    What preconditions does the code enforce that are missing?
  □ Are ensures conditions correct?
    Do they describe what the code actually guarantees?
  □ Are failure conditions exhaustive?
    Are there error conditions in the code not in the spec?
  □ Are relationships correctly represented?
  □ Are [POSSIBLE BUG] items actually bugs?
    If yes: fix the code, not the spec

□ Are there operations in the code not in the spec?
□ Are there queries in the code not in the spec?
□ Does the spec document intent or does it document bugs?
  The spec must document intent.
```

---

### Phase 2c — Generate arch.md

After all .ispec files are approved. arch.md is generated
from the full picture, not from individual domains.

```
Read "ispec system/ispec-language-spec.md" for the language rules.
Read the approved discovery report.
Read all approved .ispec files in specs/.
Read "ispec system/global.policy".

Generate arch.md for this system.

Generate each section listed below.
Mark inferences and gaps consistently.

---

# Architecture Document
## [System Name]

### System Overview
What this system is and what it does.
Who it serves.
Scale and deployment context.

### Technology Stack
List all technologies with their roles.
Note any technology choices that appear suboptimal
or that may need to change. Be honest.

### Domain Structure
For each domain in specs/:
  - What it is responsible for
  - What it exposes (operations, queries, events)
  - What it depends on
  - Approximate size and complexity

### System Boundaries
What is inside this system.
What is outside (external services, APIs, systems).
How the boundary is crossed (HTTP, message queue, database).

### Data Architecture
Where data lives.
How it flows between domains.
Consistency model (eventual vs strong).
Backup and recovery approach [INFERRED or UNKNOWN].

### Cross-Domain Communication
How domains communicate with each other.
Event-driven vs direct call patterns found.
Message queue or event bus if present.

### Infrastructure
Deployment model as found in configuration.
Scaling approach [INFERRED or UNKNOWN].
Environment structure (dev/staging/prod) if evident.

### Security Model
Authentication approach found in code.
Authorization patterns found in code.
Data sensitivity classification [INFERRED].
Known gaps identified during discovery.

### Non-Functional Characteristics
Performance characteristics [INFERRED from code patterns].
Reliability approach [INFERRED].
Observability — logging, monitoring found [describe what exists].

### Decisions Log
Document every significant decision found in the code.
For each decision:
  What: the decision that was made
  Evidence: where in the code this is visible
  Reason: [UNKNOWN unless documentation exists]
  Alternatives: [UNKNOWN unless documentation exists]
  Assessment: honest assessment of whether this was a good decision

Note: reasons and alternatives will almost always be UNKNOWN
for reverse-engineered systems. This is expected.
These gaps should be filled by the humans who made the decisions.

### Technical Debt
Honest assessment of technical debt found during discovery.
Do not soften this. Technical debt that is not named
cannot be managed.

Categories:
  Critical: must be addressed before significant new work
  Significant: should be planned for resolution
  Minor: acceptable to carry, note and monitor

### Gaps and Open Questions
Everything that could not be determined from the code.
Number each item. These require human input.

---

Mark all inferences as [INFERRED: description].
Mark all gaps as [UNKNOWN — requires human input: description].

Output to: arch.md

Stop. Present the generated file. Wait for review.
```

**arch.md review checklist:**

```
□ Stack description is accurate
□ Domain structure matches the approved .ispec files
□ System boundaries correctly identify external dependencies
□ Data architecture reflects how data actually flows
□ Security model gaps are honestly described
□ Technical debt assessment is honest — not softened
□ Decisions log gaps have been filled where possible
  (humans who made the decisions should annotate this)
□ Open questions have been answered or acknowledged
```

---

### Phase 2d — Generate context.md

Last. Context.md requires the most human input of any document.
The AI can describe what the system does. It cannot know why it was
built, who uses it, or what problem it was solving.

```
Read the approved discovery report.
Read the approved arch.md.
Read all approved .ispec files.

Generate a draft context.md for this system.

For each section, use everything found during discovery.
Be explicit about what is inferred vs confirmed.
Leave genuine gaps as structured placeholders —
do not invent information to fill them.

---

# Context Document
## [System Name]

### The Problem
[REQUIRES HUMAN INPUT]
What was broken or missing before this system existed?
Who was experiencing the problem?
What did the world look like without this system?

Draft based on discovery:
[AI-generated description of what the system appears to solve,
clearly marked as inferred]

### The Solution
What this system does about the problem.
[AI-generated from discovery and .ispec files]

What it explicitly does not do (out of scope):
[REQUIRES HUMAN INPUT — list what is known to be out of scope]
[AI can note any explicit scope boundaries found in code or docs]

### The Users
Who uses this system and how.

[For each user type identified during discovery:]
  User type: [name]
  Evidence: [what in the code suggests this user type]
  What they do: [operations available to them]
  What they care about: [INFERRED or UNKNOWN]

[REQUIRES HUMAN INPUT — correct and complete the user descriptions]

### Success Criteria
How would you know this system is working correctly?
[REQUIRES HUMAN INPUT]

Current indicators found in code (monitoring, logging, metrics):
[AI-generated from discovery]

### Constraints
Technical constraints: [from arch.md]
Business constraints: [REQUIRES HUMAN INPUT]
Regulatory constraints: [INFERRED from data handling patterns or UNKNOWN]
Timeline constraints: [UNKNOWN]

### Assumptions
What assumptions does this system appear to make?
[AI-generated from code patterns — mark all as INFERRED]

[REQUIRES HUMAN INPUT — correct these and add missing assumptions]

### Known Limitations
What does this system not do well?
[AI-generated from technical debt section of arch.md]
[REQUIRES HUMAN INPUT — add business and operational limitations]

---

Mark all inferences as [INFERRED: description].
Mark all gaps as [REQUIRES HUMAN INPUT: description].
Do not invent information. Leave gaps as gaps.

Output to: context.md

Stop. Present the generated file.
Flag all sections requiring human input clearly.
```

**context.md completion guide:**

The AI cannot fill these. They require the humans who built
or operate the system:

```
MUST BE HUMAN-FILLED
——————————————————————————————————————
The problem section
  — Only the people who experienced the problem
    or built the solution know this

Success criteria
  — Only the stakeholders know what success looks like

Business constraints
  — Budget, timeline, organizational decisions
    are not in the code

Regulatory constraints
  — Unless explicitly in the code, these require
    human knowledge

User motivations
  — What users care about cannot be inferred from code

Out-of-scope decisions
  — What was deliberately excluded requires
    knowledge of what was considered and rejected

SHOULD BE HUMAN-REVIEWED AND CORRECTED
——————————————————————————————————————
User type descriptions
  — AI inference from operations may be incomplete
    or wrong about who actually uses what

Assumption list
  — AI identifies code-level assumptions
    business assumptions are not visible in code

Known limitations
  — Technical limitations are visible
    business and operational limitations are not
```

---

## The Complete Output

When Phase 2 is complete, you have:

```
context.md              ← draft, needs human completion
arch.md                 ← draft, decisions log needs annotation
ispec system/
  global.policy         ← ready for use
  .intent               ← not generated — requires human authorship
                          see ispec system/intent.example
specs/
  domain-one.ispec      ← reviewed and approved
  domain-two.ispec      ← reviewed and approved
  ...
```

**What is not generated — .intent**

The `.intent` file is not generated by this workflow.
It cannot be. The .intent file is the constitutional layer —
the purpose, values, and permanent boundaries of the system.
These are human decisions, not things recoverable from code.

Use `ispec system/intent.example` as a template.
Fill it in with the humans who own the system.
This is not optional. A system without a .intent file
has no defined values and no defined boundaries.
That is a system that can go wrong in any direction.

---

## Legacy Codebase — Additional Guidance

For codebases that are old, poorly structured, or heavily tangled,
the standard workflow applies but with these modifications:

**During Phase 1 discovery:**

Add this to the Phase 1 prompt:

```
This is a legacy codebase. Apply additional scrutiny:

  - Note any patterns that suggest the code has been
    modified many times by different people with different
    intentions
  - Identify any code that appears to be from different
    eras (different naming conventions, different patterns)
  - Flag any dead code — functions defined but never called,
    database tables with no application references
  - Note any database schema elements that appear vestigial
  - Identify any business rules that appear to be implemented
    incorrectly based on the domain context
  - Note any places where the same concept appears to be
    implemented in multiple inconsistent ways

Add a Legacy Debt section to the discovery report listing
all of the above findings.
```

**During Phase 2 generation:**

Add this to each domain generation prompt:

```
This domain is being extracted from a legacy codebase.
When generating the .ispec:

  - Generate the spec for what the system SHOULD do,
    not necessarily what the code currently does
  - Where the code appears to implement something incorrectly,
    mark it as [POSSIBLE BUG] and generate the spec
    for the correct behavior
  - Where the same concept is implemented multiple times
    inconsistently, generate the spec for the canonical
    version and mark the inconsistency
  - Do not let legacy bugs become permanent spec definitions
```

**Human review is more important for legacy systems.**

The gap between what the code does and what it was supposed to do
is larger in legacy codebases. The review stages are where that gap
gets closed. Do not rush them.

---

## Common Patterns and How to Handle Them

### The God Class
A class or module that does everything. Common in older codebases.

```
Found: UserManager.py
  - handles authentication
  - handles profile management
  - handles billing
  - handles notifications
  - handles reporting
```

**How to handle:**
Split into separate domains in the .ispec output.
`user-account.ispec`, `billing.ispec`, `notifications.ispec`.
Note in arch.md that this refactoring is needed.
The .ispec documents the target architecture, not the current mess.

---

### The Missing Domain
Business logic scattered across many files with no clear home.

```
Found: pricing logic in:
  - order_service.py
  - cart.py
  - discount_manager.py
  - admin/pricing.py
```

**How to handle:**
Identify this as a missing domain in the discovery report.
Create `pricing.ispec` that consolidates the scattered logic.
Note in arch.md that consolidation is needed.

---

### The Implicit Contract
Operations with no validation that clearly require it.

```
Found:
  def process_payment(amount, card_number):
      charge_card(card_number, amount)
      # no validation, no authorization check
```

**How to handle:**
Generate the .ispec operation with correct requires and failures
that the code should have had. Mark as [POSSIBLE BUG].
Do not generate a spec that says authorization is not required
just because the code doesn't check it.

---

### The Undocumented Business Rule
Logic that is clearly business-specific but unexplained.

```
Found:
  if order.total > 10000 and not user.is_verified:
      raise Exception("limit")
```

**How to handle:**
Surface in Phase 1 as a question: "What is the business reason
for the 10000 limit on unverified users?"
In Phase 2, generate the spec with a named policy:
```
[policy]
  PaymentPolicy:
    unverified user maximum order: 10000
    [REQUIRES HUMAN INPUT: confirm this limit and its reason]
```

---

## Round-Trip Validation

Once the .ispec files are generated and approved, validate them
by compiling back to code and comparing:

```
1. Take approved .ispec files
2. Compile to the same language as the original codebase
3. Compare compiler output to original code

Differences reveal:
  a) Compiler gaps — the spec was right but compiler
     generated different code. Fix the compiler.

  b) Original code bugs — the spec is correct and the
     original code was wrong. This is a finding.
     Fix the original code.

  c) Spec language gaps — something in the original code
     could not be expressed in the spec language.
     Document and add to the spec language backlog.

  d) Correct differences — the compiler chose a better
     implementation than the original. Accept it.
```

This round-trip is the most powerful validation tool available.
Every difference is information. None of them are noise.

---

*Reverse .ispec System — Version 0.1 — March 2026*
*Part of the .ispec language framework*
*Copyright 2026 Christopher Rehm. All rights reserved.*
