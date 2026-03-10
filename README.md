# .ispec — A Behavioral Specification System

> *Write what your software does. AI writes the code. Humans approve the contract.*

---

## Copyright

Copyright 2026, Christopher Rehm. All rights reserved.

You may use this system to create commercial software or anything else you need.
You may not sell this system to others, fork it and sell it, or repackage it
without written permission.

Free sharing is encouraged.

If this project has helped you, consider supporting it:
[paypal.me/christopherrehm001](https://paypal.me/christopherrehm001)

---

## A Note On Working With AI

Before anything else — read this. It will save you hours.

**AI is not a mind reader. Humans assume too much. That combination is where projects fail.**

When you work with an AI to build software, the quality of what you get out is entirely
determined by the quality of what you put in. Vague instructions produce vague code.
Assumed context produces wrong assumptions in the output.

The right workflow:

1. Describe what you want — as completely as you can
2. Ask the AI to ask you clarifying questions before it does anything
3. Answer the questions fully — do not skip them
4. Only when the AI confirms it has enough information, ask it to proceed
5. Review the output carefully — you are the approval gate

The .ispec framework is designed around this discipline. The AI guides you through a
structured process. You provide the knowledge. The AI provides the structure.
Neither one can do the other's job.

If you find yourself saying "the AI got it wrong" — the first question to ask is:
"did I give it enough information?" The answer is usually no.

---

## What This Is

**.ispec** is a behavioral specification language for AI-assisted software development.

You describe what your software should do in plain English. An AI translates that into
a formal specification. A compiler turns the specification into working code in whatever
language you need.

The specification exists in two views simultaneously — plain English for humans, formal
syntax for the compiler — generated from the same source so they can never drift apart.
The client reads the plain English and approves it. The compiler reads the formal spec
and executes it. What the client approved is what gets built.

This solves a problem that has existed since the beginning of software:
**documentation always lies, because humans wrote it and humans stopped maintaining it.**
The .ispec system removes humans from the synchronization loop entirely.

---

## The Underlying Problem This Solves

**Humans assume too much.**

This is not a criticism. It is a structural fact about how humans communicate. When a
client describes what they want, they leave out everything that seems obvious to them.
When a developer builds it, they fill in the gaps with what seems obvious to them.
Those two sets of assumptions are never identical.

The result: software that does something slightly different from what was asked.
Every time. Without exception.

The .ispec framework makes assumptions explicit at every stage:

- The AI asks clarifying questions before generating anything
- The client reads and approves plain English before code is written
- The specification is the contract — not a document that shadows the code,
  but the source the code is generated from
- Every decision is recorded — not just what was built, but why

The framework does not fix the assumption problem by making humans better at
communicating. It fixes it by making assumptions visible and forcing them to be
resolved before they become bugs.

---

## The Core Idea

```
Human describes intent
        ↓
AI asks clarifying questions
        ↓
Human answers fully
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
├── README.md                              ← this file
├── CODE_OF_CONDUCT.md                     ← community standards
│
├── ispec system/                          ← the core framework
│   ├── ispec-language-spec.md             ← full language specification
│   ├── ai-guided-workflow.md              ← AI guides you through the process
│   ├── testing-strategy.md                ← what to test, why, and when
│   ├── testing-setup-guide.md             ← how to set up testing (Python/pytest)
│   ├── global.policy                      ← system-wide error hierarchy and rules
│   └── intent.example                     ← template for your .intent file
│
├── reverse ispec system/                  ← going the other direction
│   └── reverse-engineering-workflow.md    ← workflow: existing code → .ispec
│
└── examples/                              ← real .ispec files to learn from
    └── client-tracker.ispec               ← consultancy client tracker
```

---

## Quick Start

### 1. Let the AI guide you
Start with `ispec system/ai-guided-workflow.md`. The AI runs a structured
discovery process with you. You answer questions. The AI generates documents.
You approve at each gate. Nothing proceeds without your approval.

### 2. Read the language spec
`ispec system/ispec-language-spec.md` covers every construct with examples.
Read it to understand what the AI is generating on your behalf.

### 3. Look at a real example
`examples/client-tracker.ispec` shows a complete domain in both the human
view and the formal spec view.

### 4. Generate code from a spec
```
Read "ispec system/ispec-language-spec.md" for the language rules.
Read "examples/client-tracker.ispec" for the specification.
Generate Python from the [spec] section.
Output to generated/python/client_tracker.py
```

### 5. Reverse engineer existing code
```
Read "ispec system/ispec-language-spec.md" for the language rules.
Read "reverse ispec system/reverse-engineering-workflow.md" for the process.
Reverse engineer [your code path] into an .ispec file.
```

---

## The Full Framework

The .ispec language is one component of a larger framework.
A complete project requires all of these documents, generated in order,
each approved before the next is created.

| File | Layer | Purpose |
|---|---|---|
| `.intent` | 0 — Constitution | Purpose, values, what the system will never do |
| `context.md` | 1 — Context | The problem being solved and why |
| `arch.md` | 1 — Architecture | Structural decisions and the reasons behind them |
| `ispec system/global.policy` | 2 — Rules | System-wide error hierarchy and operational rules |
| `specs/domain.ispec` | 2 — Specification | Behavioral spec per domain |
| `phases.md` | 3 — Build plan | Phased delivery with test gates |
| `module/domain.md` | 4 — Interface | Token-budget-aware module contracts |

The AI generates all of these through guided conversation.
Humans approve each document before the next one is generated.
Nothing is built until the specification is approved.
Nothing proceeds to the next phase until the current phase test gate passes.

See `ispec system/intent.example` for a template to start your own `.intent` file.

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
The client approves the specification before any code is written.
What they read is what gets built.

**Documentation cannot drift.**
The human view and the formal spec are generated together.
No human maintains the sync. It is structural.

**The compiler owns implementation decisions.**
The spec says what. The compiler decides how.
Framework, database, optimization strategy — compiler concerns, not spec concerns.

**History is the default.**
Data is append-only unless explicitly marked `history: transient`.
Memory is cheap. Lost history is not recoverable.

**Errors are citizens.**
Every operation declares what can go wrong, under what conditions,
with what context, and whether the caller can retry.

**Domains are independent.**
Domains communicate through events, not direct calls.
One domain failing does not cascade. Anti-fragility is architectural.

**Extensions are the growth surface.**
The core language is small and stable.
New capabilities are added as named extensions.
Existing spec files never break.

**Assumptions are the enemy.**
Every document in this framework exists to make an assumption explicit
and force it to be resolved before it becomes a bug.
Vague inputs produce wrong outputs. The framework demands specificity.

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

The goal is a new computing paradigm: AI writes code in a purpose-built language,
humans approve intent at the specification level, and all existing programming
languages become compilation targets on the way to machine code or WebAssembly.

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

---

## Status

**Pre-prototype. Active development.**

The language is designed. The compiler is not yet built. Current workflow uses
Claude Code as the compiler — which proves the concept and informs the real
compiler design.

The spec language constructs are stable enough to write real .ispec files.
They will evolve based on implementation experience. Every gap found during
compilation becomes a language or compiler improvement.

---

## Contributing

The most valuable contributions right now:

- Writing .ispec files for real domains and reporting what the spec language
  could not express
- Running the compiler workflow and reporting what Claude Code got wrong
- Identifying constructs the language is missing
- Improving the compiler prompt based on output quality
- Testing the AI-guided workflow and reporting where the questions were
  wrong or incomplete

Open an issue for language gaps. Open a PR for spec examples and workflow improvements.

---

## Origin

This system was designed in a single brainstorming session in March 2026,
starting from the question: *what would a programming language look like if it
were optimized for AI to write rather than humans?*

The answer turned out to be less about syntax and more about the relationship
between human intent and running software — and the 50-year-old unsolved problem
of documentation that always lies.

The root cause of that problem: **humans assume too much, and there has never been
a structural mechanism to make those assumptions visible before they became code.**

The .ispec framework is that mechanism.

The full design conversation that produced this system — every decision, every
tradeoff, every construct designed from first principles — is preserved here:

[Design Session — March 2026](https://claude.ai/share/72a4b825-5d24-499c-aa0a-67a169a2a94d)

If you want to understand why something is the way it is, that conversation
is the primary source.

---

*The .ispec Language — Version 0.1 — March 2026*
*Copyright 2026 Christopher Rehm. All rights reserved.*
