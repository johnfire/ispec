# AI-Guided Project Workflow
## How the AI guides you through building a complete project

*This document is for the AI to follow and for humans to understand.*
*The AI runs this process. The human provides knowledge and approvals.*

---

## The Core Problem This Process Solves

Humans assume too much. When describing what they want built, people naturally
leave out everything that seems obvious to them. They assume the developer
shares their context, knows their business, understands their users, and
agrees on what words mean.

None of those assumptions are safe. With a human developer they cause miscommunication.
With an AI they cause confidently wrong output.

This process exists to surface and resolve assumptions before they become code.
The AI asks. The human answers. Nothing proceeds until the answers are sufficient.

---

## How This Works

The AI owns the process. The human owns the knowledge and the approvals.

```
AI asks questions
      ↓
Human answers
      ↓
AI identifies gaps and asks more questions
      ↓
Human answers
      ↓
AI generates a document
      ↓
Human reviews and approves (or requests changes)
      ↓
Next stage begins
```

Every stage has an approval gate. The AI does not proceed past a gate
without explicit human approval. "Looks fine" is not approval.
Approval means: "I have read this, it is correct, proceed."

---

## The Documents Generated — In Order

```
Stage 1   .intent          the constitution
Stage 2   context.md       the problem
Stage 3   arch.md          the structure
Stage 4   global.policy    the rules
Stage 5   domain.ispec     the specification (one per domain)
Stage 6   phases.md        the build plan
Stage 7   module/X.md      the module interfaces
```

Each document depends on the ones before it.
Do not skip stages. Do not reorder them.
A skipped stage is an assumption waiting to become a bug.

---

## Stage 1 — The Constitution (.intent)

The AI asks questions to understand the purpose and values of the system
before anything technical is discussed.

**The AI will ask:**

```
1. What does this system exist to do?
   In one sentence — what problem does it solve?

2. Who does it serve?
   List everyone who depends on it, in order of priority.
   When their needs conflict, who wins?

3. What does this system value above all else?
   What would you never sacrifice for performance or convenience?

4. What must this system never do?
   Permanent prohibitions — lines that cannot be crossed
   regardless of any other consideration.

5. If the system encounters a situation not covered by its rules,
   what principle should guide its behavior?
```

**The AI produces:** `.intent`

**Approval gate:** The human reads the .intent file and confirms:
- The purpose is stated correctly
- The stakeholder priority is right
- The values reflect what the organization actually believes
- The boundaries are complete and correctly stated

**Do not proceed until approved.**

---

## Stage 2 — The Problem (context.md)

The AI asks questions to understand the problem being solved,
not the solution. The solution comes later. Jumping to solutions
before the problem is fully understood is one of the most common
causes of software that misses the mark.

**The AI will ask:**

```
1. What is broken or missing right now?
   Describe the world without this system.
   What pain exists? Who feels it?

2. Who are the users of this system?
   Not the organization — the actual humans who will
   interact with it day to day.
   What do they know? What do they not know?
   What do they care about most?
   What frustrates them about the current situation?

3. What does success look like?
   If this system works perfectly, what is different
   six months from now?
   How would you know it is working?

4. What is explicitly out of scope?
   What problems will this system NOT solve?
   This is as important as what it will solve.

5. What constraints exist?
   Time, budget, scale, regulations, integrations
   with existing systems, technical limitations?

6. What assumptions are you making?
   What are you taking for granted that, if wrong,
   would change everything?
```

**The AI produces:** `context.md`

**Approval gate:** The human reads context.md and confirms:
- The problem description is accurate
- The users are correctly described
- Success criteria are measurable and agreed
- Out-of-scope items are explicitly listed
- Constraints are complete
- Assumptions are surfaced and accepted or challenged

**Do not proceed until approved.**

---

## Stage 3 — The Architecture (arch.md)

The AI asks questions to understand structural decisions.
This is a conversation, not a questionnaire. The AI proposes
options, explains tradeoffs, and asks the human to decide.
The human does not need to arrive with architectural knowledge.
The AI provides the options. The human provides the priorities.

**The AI will ask:**

```
1. What languages and frameworks are available or preferred?
   Is there an existing stack this must fit into?
   Are there languages or tools explicitly off the table?

2. How will this system be deployed?
   Local machine, server, cloud, mobile?
   Who operates it? What is their technical level?

3. What are the scale requirements?
   How many users? How much data? What response time is acceptable?
   Does this need to grow significantly?

4. What does this system integrate with?
   Other systems, APIs, databases, services?
   Which of those integrations are critical vs optional?

5. What are the security requirements?
   Who should have access to what?
   Is there sensitive data? Regulatory requirements?

6. What is the reliability requirement?
   What happens if this system is unavailable?
   Is downtime acceptable? For how long?
```

**The AI then proposes an architecture** based on the answers,
explaining each decision and its tradeoffs. The human challenges,
modifies, or approves each decision. Every decision is recorded
with its reason — not just what was chosen, but why, and what
was considered and rejected.

**The AI produces:** `arch.md`

**Approval gate:** The human reads arch.md and confirms:
- Stack decisions are correct and understood
- System boundaries make sense
- The decisions log accurately records what was decided and why
- Non-functional requirements are captured

**Do not proceed until approved.**

---

## Stage 4 — The Rules (global.policy)

The AI generates global.policy from the context and architecture.
This stage is mostly generative — the AI produces the file and
the human reviews it for correctness.

The AI will ask only targeted questions:

```
1. Are there domain-specific retry requirements?
2. Are there regulatory requirements around data retention?
3. Are there specific rate limiting or pagination requirements?
4. Are there global audit requirements?
```

**The AI produces:** `global.policy`

**Approval gate:** Human reviews and confirms the error hierarchy
and global rules are correct for this system.

---

## Stage 5 — The Specification (domain.ispec)

One domain at a time. Do not attempt to specify the whole system
at once. Identify the domains first, then specify each one.

**Step 5a — Domain identification**

The AI asks:
```
1. What are the major areas of concern in this system?
   Examples: user accounts, payments, orders, notifications.
   Each major area is probably a domain.

2. How do these areas communicate?
   When something happens in one area, what other areas need to know?

3. Which domain should we specify first?
   Usually: the one everything else depends on.
   Often: user accounts or the core business entity.
```

**Step 5b — Domain specification (per domain)**

For each domain, the AI conducts a structured interview:

```
ENTITIES
  What things does this domain store and reason about?
  For each thing:
    What information does it have?
    What states can it be in?
    Does it need full history or can it be overwritten?
    How does it relate to other things?

OPERATIONS
  What does this domain do?
  For each action:
    Who triggers it?
    What must be true before it can happen?
    What is guaranteed to be true after it succeeds?
    What can go wrong?
    Does anything else need to know when it happens?

QUERIES
  What questions does this domain answer?
  For each query:
    What is being asked?
    What filters apply?
    What order should results be in?

RULES
  What business rules apply to this domain?
  What are the limits and policies?
```

**The AI produces:** `specs/domain-name.ispec`
with both [human] and [spec] sections.

**Approval gate — TWO REVIEWERS:**
- Non-technical reviewer reads [human] section only
  confirms: this describes what we want correctly
- Technical reviewer reads [spec] section
  confirms: this is implementable and complete

**Do not proceed to the next domain until this one is approved.**

---

## Stage 6 — The Build Plan (phases.md)

With all domains specified, the AI proposes a phased build plan.

**The AI asks:**
```
1. What is the minimum useful version of this system?
   What is the smallest thing that delivers real value?
   That is Phase 1.

2. What must work before anything else can be built on top of it?
   That is probably Phase 1 as well.

3. What can wait until Phase 2 or later?

4. What are your testing capabilities?
   Unit tests, integration tests, manual QA?
   Who does the testing?
```

**The AI produces:** `phases.md`

Structure of each phase:
```
Phase N
  Goal:         one sentence
  Includes:     named constructs from .ispec files
  Excludes:     explicitly not in this phase
  Test gate:    what must pass — specific and measurable
  Acceptance:   what a human verifies before sign-off
  Depends on:   which previous phases must be complete
```

**Approval gate:** Human confirms the phase breakdown is realistic,
the test gates are achievable, and Phase 1 delivers real value.

**The test gates are non-negotiable.** Phase N+1 does not start
until Phase N test gate passes. The AI will not proceed past a
failed test gate. This is the single most important discipline
in the build process. Skipping it means building on a broken
foundation. The failure will compound with every phase.

---

## Stage 7 — Module Interfaces (module/domain.md)

For each domain, the AI generates a module interface document.
This is what other modules read when they need to interact with
this one — they do not read the full code. They read the interface.

This keeps each module within a manageable token budget for the AI.
When working on payments, the AI reads the payments interface,
not the full user-account codebase.

**The AI produces:** `module/domain-name/README.md`

Contents:
```
What this module does (one paragraph)
What it exposes (operations, queries, events)
What it depends on (use statements from .ispec)
What it emits (events other modules subscribe to)
Error types it produces
Token budget status (estimated size vs limit)
```

---

## The Build Discipline

Once all documents are approved, the build follows this discipline
for every piece of work:

```
BEFORE WRITING ANY CODE
  1. Read .intent
  2. Read context.md
  3. Read arch.md
  4. Read the relevant domain.ispec
  5. Read the relevant module interface
  6. Check phases.md — is this work in the current phase?
     Is the previous phase test gate passed?

WRITING CODE
  7. Generate code from the [spec] section of the .ispec
  8. Follow arch.md decisions for framework and stack choices
  9. Stay within the module boundary
  10. Do not reach into another module's code — use its interface

AFTER WRITING CODE
  11. Run the phase test gate
  12. Do not proceed until it passes
  13. If it fails, fix the code — not the test gate
  14. Record any spec language gaps found during compilation
```

---

## When The AI Gets It Wrong

If the AI produces something incorrect, the first question is always:
**was the input sufficient?**

Common causes of wrong output:

```
SYMPTOM                          LIKELY CAUSE
AI misunderstood the domain      Stage 2 questions were not answered
                                 fully enough

AI made wrong architectural      Stage 3 tradeoffs were not discussed
choices                          or the human did not correct the
                                 AI's proposal

.ispec does not match            Stage 5 interview was rushed or
what was intended                questions were answered vaguely

Code does not match spec         The .ispec has a gap — add a
                                 compiler rule and document it

Phase is failing test gate       Do not proceed. Fix the issue.
                                 Do not adjust the test gate.
```

Every wrong output is information. Record what was wrong,
what caused it, and what was done to fix it. That record
improves every future project.

---

## Prompt Template — Starting A New Project

Copy this into Claude Code or Claude to begin:

```
Read ispec-language-spec.md for the .ispec language rules.
Read ai-guided-workflow.md for the process you will follow.

You are going to guide me through building a new software project.
Follow the ai-guided-workflow.md process exactly.
Ask questions one stage at a time.
Do not generate any document until you have enough information.
Do not proceed past any approval gate without my explicit approval.
Do not assume. Ask.

When you are ready to begin, start Stage 1.
Ask me the Stage 1 questions one at a time.
Wait for my answers before asking the next question.
```

---

*AI-Guided Workflow — Version 0.1 — March 2026*
*Part of the .ispec language framework*
*Copyright 2026 Christopher Rehm. All rights reserved.*
