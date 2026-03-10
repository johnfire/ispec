# Testing Strategy
## What to test, why, and when — for .ispec projects

*This document is framework guidance. It applies to any project built
using the .ispec system regardless of language or stack.*

---

## The Core Principle

**Tests are the proof that the spec was satisfied.**

The .ispec system makes a specific promise: the client approved the
specification, the compiler generated code from the specification,
and the running system does what the specification says.

Tests are how you verify that promise is kept. Without them you have
a spec, some code, and hope. Hope is not a quality assurance strategy.

---

## The Second Principle

**When a test fails, fix the code. Never fix the test.**

This rule has no exceptions. A failing test is information. It is telling
you that the code does not satisfy the contract. The correct response is
to fix the code. Adjusting the test to make the failure go away is not
a fix — it is a lie that will surface in production at the worst possible
moment.

This applies to test gates between phases. If the gate fails, the phase
is not complete. Fix the code. Do not lower the gate.

---

## The Six Testing Layers

### Layer 1 — Unit Tests

**What:** Individual operations and queries tested in isolation.
Each operation is called with valid inputs and the outputs are verified.
Each failure condition is triggered and verified to produce the correct error.

**Generated from:** `.ispec` operation `ensures` and `failures` sections.
The compiler should generate the test scaffolding automatically.
The developer fills in fixture data.

**When runs:** Every commit. Fast. Should complete in seconds.

**Gate:** Phase test gate — must pass before the next phase begins.

**What a unit test covers:**
```
For operation TransferFunds:
  ✓ valid transfer reduces source balance correctly
  ✓ valid transfer increases destination balance correctly
  ✓ transfer is atomic — both change or neither changes
  ✓ InsufficientFunds raised when balance < amount
  ✓ NotAuthorized raised when actor != source.owner
  ✓ transfer with amount == 0 raises ValidationError
  ✓ transfer with negative amount raises ValidationError
```

---

### Layer 2 — Integration Tests

**What:** Modules working together. Events emitted by one domain
received correctly by another. Database operations persisting and
retrieving correctly. Cross-domain references resolving correctly.

**Generated from:** `.ispec` cross-domain `use` statements and
`on Event` subscriptions.

**When runs:** Every commit. Slower than unit tests. May require
a test database.

**Gate:** Phase test gate.

**What an integration test covers:**
```
For event FundsTransferred → SendTransferConfirmation:
  ✓ completing a transfer emits FundsTransferred event
  ✓ notification domain receives the event
  ✓ confirmation is triggered with correct data
  ✓ if notification fails, transfer is not rolled back
  ✓ duplicate event delivery is handled idempotently
```

---

### Layer 3 — End-to-End Tests

**What:** Complete user workflows from entry point to final state.
Real HTTP requests, real database, real event handling. Tests the
system as a user experiences it, not as a developer tests it.

**Generated from:** Complete operation sequences that represent
real user journeys. These are not generated automatically —
they require human knowledge of how the system is actually used.

**When runs:** Before every release. Slower — may take minutes.

**Gate:** Release gate.

**What an end-to-end test covers:**
```
User journey: new client onboarding
  ✓ create client record
  ✓ add first project
  ✓ add a note to the project
  ✓ update project status
  ✓ verify status change note was auto-created
  ✓ query active work shows the project
```

---

### Layer 4 — Contract Tests

**What:** Direct verification that compiler output satisfies
.ispec contracts. This is the layer specific to .ispec projects
and the one most likely to be skipped. Do not skip it.

The .ispec system's value proposition is that the spec is the
contract. Contract tests are the proof. Without them you cannot
claim the system does what the spec says.

**Generated from:** `.ispec` `ensures` and `failures` sections,
with specific focus on:
- `was()` values — are pre-operation values correctly captured?
- `atomic:` constraints — do partial failures roll back correctly?
- `history: immutable` — are records appended, never overwritten?
- `history: transient` — are records overwritten, not appended?
- Aggregate correctness — does `count()`, `sum()`, `avg()` produce
  correct values after operations that change the data?

**When runs:** Every compile. If contract tests fail, the build
is broken by definition — the compiler output does not satisfy
the spec it was compiled from.

**Gate:** Compilation gate. Harder than a phase gate. Non-negotiable.

**What a contract test covers:**
```
For entity User with history: immutable:
  ✓ updating email creates a new record
  ✓ old email record still exists in history
  ✓ current state query returns the latest record
  ✓ history query returns all records in order

For operation TransferFunds with atomic: [source.balance, destination.balance]:
  ✓ simulate failure after source deducted but before
    destination credited — verify both are unchanged
  ✓ verify was(source.balance) captured before, not after
```

---

### Layer 5 — Security Testing

**What:** Deliberate attempts to find security vulnerabilities.
Covers both automated scanning and structured manual review.

Three levels — choose based on project risk:

**Level 1 — Automated static analysis**
Run on every commit. Catches known vulnerability patterns,
dependency issues, and common misconfigurations.
Tools: language-specific static analyzers, dependency scanners.
Required for: every project.

**Level 2 — .ispec security review**
AI-driven review of every `.ispec` file before compilation.
Checks for authorization gaps, data exposure, missing failure
modes, and logic vulnerabilities. Run against the spec, not
the code — catches problems before they are compiled.
Required for: every project.

**Level 3 — Penetration testing**
Human specialists conducting structured attacks against the
running system. Finds what automated tools miss — business
logic vulnerabilities, chained attack vectors, authorization
bypass through legitimate operations.
Required for: any system handling financial data, health data,
personal data at scale, or with regulatory requirements.
Recommended for: any client-facing system before public launch.

**Gate:** Release gate for Level 1 and 2. Level 3 gate defined
per project based on risk.

---

### Layer 6 — Resilience Testing

**What:** Verification that anti-fragility is real, not assumed.
The .ispec system is designed so that if one component fails,
the rest continues. Resilience testing verifies this is actually
true in the compiled output.

**When runs:** Before major releases and after significant
architectural changes.

**Gate:** Release gate for production systems.

**What resilience testing covers:**
```
Domain isolation:
  ✓ kill the notification service — do transfers still work?
  ✓ database connection lost — does the system degrade
    gracefully or crash entirely?

Event handling:
  ✓ corrupt a message in the queue — does the handler
    survive and continue processing other messages?
  ✓ flood the event queue — does the system slow down
    gracefully or fail catastrophically?

Retry behavior:
  ✓ simulate transient database failure — does retry
    logic fire correctly?
  ✓ simulate permanent failure — does retry logic
    give up after the correct number of attempts?
```

---

## The .ispec Security Review

Before any .ispec file is compiled, run this review.
The AI performs it against the spec — not the generated code.

**For every operation, verify:**

```
Authorization:
  □ Is there an authorization check in requires?
    If not: is this intentionally public? Document why.
  □ Does AuthorizationError appear in failures?
  □ Can the actor field be spoofed or omitted?

Input validation:
  □ Are all inputs validated in requires?
  □ What happens with null inputs?
  □ What happens with inputs at boundary values?
    (zero, negative, maximum length, empty string)
  □ Are ValidationError conditions exhaustive?

Data exposure:
  □ Does returns expose data the caller should not have?
  □ Can the operation be called on another user's entities?
  □ Does the operation expose internal identifiers?

Side effects:
  □ Can side effects be triggered without completing
    the main operation?
  □ Are events carrying sensitive data that should
    not leave the domain?
```

**For every query, verify:**

```
Scope:
  □ Can a user query another user's data?
  □ Are all where conditions necessary and sufficient?
  □ Can the query be used to enumerate all records
    when it should only return the caller's records?

Data exposure:
  □ Does the query return fields that should be
    restricted? (write-only fields, internal fields)
  □ Is pagination enforced? Can a caller retrieve
    the entire dataset in one request?
```

**For every event, verify:**

```
Source validation:
  □ Is the event source authenticated?
  □ Can an external actor inject a fake event?
  □ Does the event carry data that should not
    leave the originating domain?

Handler safety:
  □ Is the event handler idempotent?
  □ What happens if the handler receives a
    malformed event?
```

---

## Penetration Testing — Briefing Guide

If your project requires Level 3 penetration testing, the .ispec
documents are an unusually complete brief for the testing team.
Use them.

**What to give the penetration tester:**

```
.intent          — boundaries the system must never cross
                   tells the tester what a successful attack looks like

context.md       — who the users are and what they can do
                   tells the tester what normal behavior looks like

domain.ispec     — every operation, every query, every event
  [human] section — plain English — testers can read this directly
  authorization model — who can do what

arch.md          — infrastructure, integrations, deployment
                   tells the tester the attack surface
```

**What to ask the penetration tester to focus on:**

```
1. Authorization bypass
   Can an actor perform an operation they are not permitted?
   Can requires conditions be bypassed?

2. Data extraction
   Can a user retrieve another user's data through queries?
   Can events be used to extract data across domain boundaries?

3. Injection
   Can inputs be crafted to cause unexpected behavior?
   SQL injection, command injection, XSS in any text fields?

4. Business logic attacks
   Can operations be sequenced in unexpected ways?
   Can race conditions be exploited in atomic operations?
   Can the retry mechanism be abused?

5. Event system attacks
   Can fake events be injected?
   Can events be replayed maliciously?
```

---

## Test Gates — Reference

```
GATE              WHEN                  WHAT MUST PASS
——————————————————————————————————————————————————————
Compilation gate  Every compile         Contract tests
                                        Static analysis (Level 1)

Commit gate       Every commit          Unit tests
                                        Integration tests
                                        Static analysis (Level 1)

Phase gate        End of each phase     All commit gate tests
                                        Phase-specific acceptance
                                        Human sign-off

Release gate      Before any release    All phase gate tests
                                        End-to-end tests
                                        .ispec security review (Level 2)
                                        Penetration testing (Level 3
                                          where required)
                                        Resilience testing (where required)
```

---

## The Non-Negotiable Rules

**Fix the code, never the test.**
A failing test is information. The correct response is to fix the code.

**Test gates do not move.**
A phase is not complete until its gate passes. Proceeding on a failed
gate means building on a broken foundation. The failure compounds.

**Security review happens before compilation.**
Finding an authorization gap in the spec costs nothing to fix.
Finding it after deployment costs everything.

**Penetration testing findings are fixed, not accepted.**
"The attack vector is unlikely" is not a fix. Fix the vulnerability.

**Contract tests are not optional.**
They are the proof that the .ispec system's core promise is kept.
Without them the spec and the code are uncoupled. That is not better
than what existed before.

---

*Testing Strategy — Version 0.1 — March 2026*
*Part of the .ispec language framework*
*Copyright 2026 Christopher Rehm. All rights reserved.*
