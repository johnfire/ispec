# .ispec — A Behavioral Specification System

> *Write what your software does. AI writes the code. Humans approve the contract.*

---

## What This Is

**.ispec** is a behavioral specification language for AI-assisted software development.

You describe what your software should do in plain English. An AI translates that into a formal specification. A compiler turns the specification into working code in whatever language you need.

The specification exists in two views simultaneously — plain English for humans, formal syntax for the compiler — generated from the same source so they can never drift apart. The client reads the plain English and approves it. The compiler reads the formal spec and executes it. What the client approved is what gets built.

This solves a problem that has existed since the beginning of software: **documentation always lies, because humans wrote it and humans stopped maintaining it.** The .ispec system removes humans from the synchronization loop entirely.

---

## The Core Idea

```
Human describes intent
        ↓
AI generates .ispec file
  [human]  ← plain English  — client reads this
  [spec]   ← formal syntax  — compiler reads this
        ↓
Human approves
  (this approval is the contract)
        ↓
Compiler generates code
        ↓
Running system
  [human] section is now living documentation
  forever in sync — automatically
```

---

## Repository Contents

```
.
├── README.md                         ← this file
├── ispec-language-spec.md            ← full language specification
├── reverse-engineering-workflow.md   ← workflow: code → .ispec
├── specs/
│   └── client-tracker.ispec         ← example: consultancy client tracker
└── global.policy                     ← system-wide error hierarchy and rules
```

---

## Quick Start

### 1. Read the language spec
Start with `ispec-language-spec.md`. It covers every construct with examples.

### 2. Look at a real example
Open `specs/client-tracker.ispec`. It shows a complete domain — entities, operations, queries, events, and extensions — in both the human view and the formal spec view.

### 3. Generate code from a spec
Give Claude Code both files and ask it to compile:

```
Read ispec-language-spec.md for the language rules.
Read specs/client-tracker.ispec for the specification.
Generate Python from the [spec] section.
Output to generated/python/client_tracker.py
```

### 4. Reverse engineer existing code
Use `reverse-engineering-workflow.md` to turn an existing codebase into an .ispec file.

```
Read ispec-language-spec.md for the language rules.
Read reverse-engineering-workflow.md for the process.
Reverse engineer [your code path] into an .ispec file.
```

---

## The File Types

| File | Purpose |
|---|---|
| `.intent` | System constitution — purpose, values, what the system will never do |
| `global.policy` | System-wide rules — error hierarchy, retry policy, pagination defaults |
| `domain.ispec` | Domain specification — entities, operations, queries, events |

### Inside an .ispec file

```
[human]
  Plain English. Non-technical readers verify this.
  Clients approve this before anything is built.
[/human]

[policy]
  Domain-specific rules. Override global policies where needed.
[/policy]

[spec]
  Formal specification. Compiler reads this.
  Entities, operations, queries, events, extensions.
[/spec]
```

---

## The Spec Language — At A Glance

### Entities
```
entity User
  id:     UserId  [unique, generated, immutable]
  email:  Email   [unique, required]
  status: Active | Suspended | Deleted
```

### Operations
```
operation TransferFunds
  inputs:
    source:      Account
    destination: Account
    amount:      Money

  requires:
    source.balance >= amount

  ensures:
    source.balance      == was(source.balance) - amount
    destination.balance == was(destination.balance) + amount
    atomic: [source.balance, destination.balance]

  failures:
    InsufficientFunds: BusinessError
      condition: source.balance < amount
```

### Queries
```
@query GetActiveUsers
  find:  User
  where: User.status == Active
  order: User.created_at descending
```

### Events
```
event FundsTransferred
  source:         → Account  [required]
  destination:    → Account  [required]
  amount:         Money      [required]
  transferred_at: Timestamp  [required]
```

### Cross-domain references
```
use User from user-account
use Product from products
```

### Aggregates
```
count(Enrollment where Enrollment.course == course
      and Enrollment.status == Active)

sum(OrderItem.unit_price * OrderItem.quantity
    where OrderItem.order == order)

exists(Session where Session.user == user
       and Session.status == Valid)
```

---

## Design Principles

**The spec is the contract.**
The client approves the specification before any code is written. What they read is what gets built.

**Documentation cannot drift.**
The human view and the formal spec are generated together. No human maintains the sync. It is structural.

**The compiler owns implementation decisions.**
The spec says what. The compiler decides how. Framework, database, optimization strategy — compiler concerns, not spec concerns.

**History is the default.**
Data is append-only unless explicitly marked `history: transient`. Memory is cheap. Lost history is not recoverable.

**Errors are citizens.**
Every operation declares what can go wrong, under what conditions, with what context, and whether the caller can retry.

**Domains are independent.**
Domains communicate through events, not direct calls. One domain failing does not cascade. Anti-fragility is architectural.

**Extensions are the growth surface.**
The core language is small and stable. New capabilities are added as named extensions. Existing spec files never break.

---

## The Error Hierarchy

```
SystemError          infrastructure problems — always retriable
  DatabaseError
  NetworkError
  TimeoutError
  ServiceUnavailable

ClientError          caller did something wrong — never retriable
  ValidationError
  AuthorizationError
  NotFoundError
  ConflictError

BusinessError        business rule violated — never retriable
  PolicyViolation
  LimitExceeded
  StateError
```

---

## The Long-Range Vision

The .ispec system is Phase 1 of a larger vision.

The goal is a new computing paradigm: AI writes code in a purpose-built language, humans approve intent at the specification level, and all existing programming languages become compilation targets on the way to machine code or WebAssembly.

```
Phase 1 (now)
  .ispec → Python / JavaScript via Claude Code
  Prove the model. Ship real software.

Phase 2
  Deterministic local compiler
  Multiple language targets
  AI in the spec generation pipeline only

Phase 3
  AI-native intermediate representation
  Dense, semantically rich, target-agnostic
  Existing languages become irrelevant

Phase 4
  IR compiles directly to machine code / WebAssembly
  One language. AI writes it. Humans approve intent.
```

The .ispec system is the foundation. Every design decision made here — the separation of intent from spec from implementation, the dual representation, the constitutional .intent layer — is coherent with where this is going.

---

## Status

**Pre-prototype. Active development.**

The language is designed. The compiler is not yet built. Current workflow uses Claude Code as the compiler — which proves the concept and informs the real compiler design.

The spec language constructs are stable enough to write real .ispec files. They will evolve based on implementation experience. Every gap found during compilation becomes a language or compiler improvement.

---

## Contributing

This is an early-stage research and implementation project. The most valuable contributions right now are:

- Writing .ispec files for real domains and reporting what the spec language couldn't express
- Running the compiler workflow and reporting what Claude Code got wrong
- Identifying constructs the language is missing
- Improving the compiler prompt based on output quality

Open an issue for language gaps. Open a PR for spec examples.

---

## Origin

This system was designed in a single brainstorming session in March 2026, starting from the question: *what would a programming language look like if it were optimized for AI to write rather than humans?*

The answer turned out to be less about syntax and more about the relationship between human intent and running software — and the 50-year-old unsolved problem of documentation that always lies.

---

*The .ispec Language — Version 0.1 — March 2026*
