# Lithify — Requirements, Invariants, and Architectural Implications

> Working product specification derived from the accepted Lithify product intent.
>
> This document intentionally starts from product intent and derives requirements
> and invariants before fixing implementation mechanisms. Terms such as backend
> interface, protocol, storage engine, process model, daemon, CLI, API syntax,
> concrete receipt schema, and programming language are deliberately left
> unspecified unless the product contract requires them.
>
> Section 0 is the normative product authority for Lithify. Sections that follow
> it may derive from it and MUST NOT weaken it. Section 2 is explicitly
> non-normative, and Section 3 records questions this specification deliberately
> leaves open.

# 0. Product intent — governing user experience

This section is normative product authority. It records the product decisions
accepted by the product owner and states the product outcome that every
lower-level requirement, architectural implication, and implementation exists to
serve. If a lower-level rule admits several interpretations, the interpretation
that preserves this product intent is required. If a mechanism cannot preserve
this intent, the mechanism is wrong, not the intent.

A change that weakens any promise in this section requires an explicit recorded
product decision that amends this section in the same change. A weakening MUST
NOT appear as a side effect of implementation, convenience, or a
backend-specific limitation.

## 0.1 Product definition: a reliable mechanical boundary for durable engineering work state

Lithify is a **reliable actuator/reconciler for durable engineering work state**.

Lithify exists to give autonomous systems a **reliable, backend-independent
mechanical boundary** for two things:

```text
establishing durable engineering work state
proving whether a requested durable engineering work state was actually reached
```

The governing distinction is:

> A successful Lithify operation means that the requested durable state was
> independently observed to hold, not merely that a backend accepted a mutation
> request.

Lithify is therefore not a richer wrapper around a work-management API, and not a
second ticket manager. Existing ticketing and engineering-work systems already
solve much of the multi-backend ticket-management problem. What they do not
provide — and what Lithify is — is a boundary whose success claim is a verified
backend fact rather than an accepted request.

Lithify is intended primarily for unattended machine callers:

- autonomous agents;
- coding-agent harnesses;
- workflow engines;
- orchestration systems;
- schedulers;
- other software systems.

Humans may invoke Lithify for diagnostics, inspection, or debugging, but normal
operation MUST NOT require human interpretation, confirmation, or supervision.
The runtime interface is therefore machine-first, not human-first, and the core
invariant is:

> **Lithify never relies on human interpretation to determine whether an
> operation succeeded.**

## 0.2 The fundamental operation

The fundamental operation is conceptually:

```text
caller determines desired engineering work state
        ↓
Lithify receives that explicit desired state
        ↓
Lithify resolves the exact external target
        ↓
Lithify observes current backend reality
        ↓
Lithify performs only the required mechanical mutation
        ↓
Lithify independently re-observes backend reality
        ↓
Lithify determines whether the requested postcondition is actually true
        ↓
machine-verifiable result / receipt
```

Two properties of this sequence are product-defining:

```text
the caller supplies the desired state
Lithify supplies the mechanics and the proof
```

This section establishes the operation model as **request-scoped actuation**: a
desired state is supplied explicitly by a caller, and Lithify attempts to
establish and prove exactly that state. Whether Lithify additionally owns any
continuous, scheduled, or repeated reconciliation loop over previously requested
state is not established by this section and remains an open product decision
(Section 3).

The operation is mechanical in the sense that Lithify's responsibility is
resolving targets, observing reality, actuating, re-observing, and reporting. It
is not mechanical in the sense of being trivial; the guarantees in this section
are the product.

## 0.3 The product promise

> **An autonomous system may give Lithify an explicit desired engineering work
> state without reasoning about the mechanics of the backing work-management
> system. Lithify resolves the exact backend target, observes current durable
> state, performs only the required mechanical actuation, re-observes backend
> reality, and reports success only when the requested postcondition is proven.
> Lithify does not decide what work state should exist, and ordinary operation
> requires no human interpretation. Backend-specific mechanics remain
> implementation details beneath this product boundary.**

The promise decomposes into the following normative requirements:

```text
1. an autonomous caller states the durable engineering work state it wants
2. Lithify owns the mechanical interaction required to attempt to establish that state
3. provider-specific mechanics remain below the Lithify semantic boundary
4. success requires a fresh observation proving the requested postcondition
5. ambiguity or inability to prove the state fails closed
6. Lithify does not choose the desired state
7. normal operation requires no human interpretation
```

A reported success that was not established by requirement 4 is a product defect,
not an implementation tradeoff.

## 0.4 Desired state versus policy

Lithify does not decide what the engineering work state should be. The caller
owns policy and semantic decisions.

Decisions outside Lithify include, non-exhaustively:

- whether work should exist;
- what work should be done;
- whether a discovered concern deserves a durable ticket;
- which item should be prioritized;
- whether an item is ready;
- whether an item is complete;
- which agent should receive a job;
- whether one work item semantically depends on another;
- whether a finding is valid;
- what workflow should execute next.

Lithify MAY mechanically establish any of those states when they are explicitly
requested by an authorized caller. For example:

```text
assign work item 42 to agent Y
```

may be a Lithify operation, while:

```text
choose the best agent for work item 42
```

is not a Lithify decision. The governing rule is:

> **Lithify decides how to establish and verify an explicitly requested state. It
> does not decide which state ought to be requested.**

Authorization is a precondition on the request, not a semantic judgment Lithify
performs on the work. What constitutes an authorized caller, and how that
authority is established, is not fixed by this section (Section 3).

## 0.5 The product domain: durable engineering work state

The product domain is durable engineering work state. It may include concepts
such as:

- durable work items;
- workflow state;
- classification;
- priority;
- assignment;
- dependencies / blockers;
- parent-child work relationships;
- linkage to execution or change artifacts;
- other durable state required by engineering-work systems.

This list MUST NOT be frozen into an implementation-specific universal schema.
The exact common semantic model MUST be derived from real backend requirements
rather than imagined prematurely. A concept belongs to Lithify's vocabulary
because concrete backend evidence requires it, not because one provider happens
to expose it.

## 0.6 Backend independence: GitHub is the initial backend, not the product

GitHub is the initial backend.

GitHub is **not** part of Lithify's fundamental product semantics. Potential
future backends may include systems such as:

- GitHub Issues / Projects;
- Linear;
- Jira;
- GitLab;
- other external engineering-work systems;
- internal durable work-state systems.

The dependency direction is normative:

```text
Lithify product semantics
        ↓
backend-independent work-state operation
        ↓
backend realization
        ↓
GitHub / Linear / Jira / ...
```

Backend-specific identifiers, APIs, GraphQL models, Project V2 node IDs, field
IDs, transition IDs, or equivalent provider mechanics MUST NOT define Lithify's
fundamental semantics.

Backend independence is a property of the semantic boundary, not a claim about
code structure. A Lithify deployment that currently speaks only to GitHub is
conformant. A Lithify whose product semantics are defined in terms of GitHub
objects is not.

This section does not authorize a speculative universal backend abstraction. Only
GitHub currently provides concrete evidence. The product MUST preserve backend
independence at the semantic boundary while the first implementation remains
concretely GitHub-driven. A real common abstraction SHOULD be extracted only when
additional backend evidence demonstrates what is actually common; an abstraction
not derived from at least two concrete backend realizations MUST NOT be frozen as
product semantics.

## 0.7 Postcondition-driven success: observed state, not accepted requests

A backend saying:

```text
200 OK
mutation accepted
request completed
```

is not by itself sufficient proof of Lithify success. Lithify MUST reason from
freshly observed durable state at the backend boundary.

The postcondition is the authoritative success criterion. Conceptually, an
operation may yield outcomes such as:

```text
NOOP
requested state was already observed

APPLIED
a mutation was required and the requested state was subsequently observed

FAILED
the requested state could not be proven
```

Those exact names, and any specific receipt schema, are not fixed implementation
API unless they are necessary at the product-semantic level. The required
semantic distinction is:

- already true;
- made true and verified;
- not proven true.

This distinction is normative:

- the "already true" determination MUST itself be an observation of current
  backend reality; a caller's assertion that state already holds is not evidence;
- an operation that requires no mutation is successful when the requested state
  is observed to hold; "nothing was changed" is not failure;
- a mutation that a backend accepted but whose postcondition was not observed
  MUST NOT be reported as success;
- success MUST NOT be reported on the basis of a response code, a mutation
  identifier, a job or transition status, or any other proxy for the durable
  state rather than the observed durable state itself.

How Lithify treats backends whose durable state materializes asynchronously, and
whether a still-materializing operation is a legitimate product-level outcome,
is not fixed by this section (Section 3).

## 0.8 Re-observation: the mutation response is not verification evidence

Verification MUST be based on a fresh observation after mutation. The response
returned by the mutation request itself is not automatically equivalent to
independent evidence of the durable resulting state.

This distinction is fundamental to Lithify. The system exists specifically so
that unattended callers do not need to infer:

```text
"the API call looked successful, therefore the desired state probably exists"
```

Instead they can depend on:

```text
"Lithify reports success only because the requested state was subsequently observed."
```

Normatively:

- verification evidence MUST come from an observation performed after the
  mutation attempted the change;
- that observation MUST be of durable backend state, not of the mutation
  request's own result envelope;
- "fresh" means the observation reflects backend reality at the time it is
  taken; a pre-mutation observation MUST NOT be reused as evidence of a
  post-mutation state;
- an "already true" determination MUST rest on an observation taken for that
  determination or on a still-current observation, not on an assumption carried
  over from the caller;
- the product MUST NOT collapse "the mutation was accepted" and "the
  postcondition was observed" into one undifferentiated success signal.

Which observation paths count as admissible independent evidence, and what
satisfies this requirement for a backend whose only post-write read is not
distinguishable from the write acknowledgement, is not fixed by this section
(Section 3).

## 0.9 Exact identity and fail-closed ambiguity

Autonomous unattended mutation requires exact targeting. Lithify MUST NOT rely on
fuzzy or human interpretation to decide which external object a mutation
concerns.

Where exact target identity cannot be established safely, Lithify fails closed
rather than guessing. Normatively:

- every mutating operation MUST be bound to exactly one resolved external target
  before the mutation is attempted;
- the target MUST be resolved from current backend reality rather than from a
  persisted mechanical identity whenever a stale persisted identity could
  invalidate correctness;
- ambiguity between two or more plausible targets is not mutation authority:
  Lithify MUST NOT select among them by heuristic, first match, recency, naming
  similarity, or any other non-authoritative rule;
- a partially resolved target — for example, an object whose containing scope
  resolves while a field, option, or state mapping remains ambiguous — MUST be
  treated as unresolved;
- unresolved identity MUST be surfaced to the caller as a not-proven outcome
  rather than silently resolved, silently retried until it happens to become
  unique, or escalated to a human during ordinary operation.

This requirement concerns the identity of the external target, not the semantics
of the requested work. A boundary that forwards a caller-supplied string into a
backend search-and-mutate call has not provided exact targeting.

## 0.10 Retry, resumption, and unattended concurrency

Lithify is intended to be safe as a primitive inside larger autonomous systems
that may:

- retry after failure;
- crash and resume;
- execute concurrently;
- lose local context;
- operate for long periods without a human watching;
- perform many engineering-work mutations automatically.

Therefore the Product Intent requires that:

- observable results distinguish verified state from uncertain state;
- where backend behavior prevents certainty, Lithify exposes uncertainty or
  failure rather than manufacturing confidence;
- Lithify MUST NOT promise universal exactly-once effects unless that can
  actually be derived and implemented against the backend;
- correctness of a re-attempt MUST NOT depend on the caller having retained local
  context, having observed a previous result, or knowing whether a previous
  attempt reached the backend;
- a re-attempt MUST establish current state by observation before actuating, and
  where the requested effect is not idempotent at the backend, the resulting risk
  MUST surface as uncertainty rather than be hidden;
- concurrent or interleaved operation MUST NOT be permitted to weaken the
  observed-postcondition requirement of Section 0.7 and Section 0.8.

This section does not fix a concurrency-control, serialization, coalescing, or
claim model (Section 3).

## 0.11 Machine-first operation and human surfaces

Lithify MAY expose human-accessible diagnostics, inspection commands, debugging
tools, or administrative interfaces. Those are secondary surfaces.

A conforming runtime MUST NOT require a human to:

- interpret whether a mutation succeeded;
- choose among ambiguous backend identities;
- inspect the backend to confirm ordinary success;
- manually repair normal backend representation differences;
- translate every operation into provider-specific mechanics;
- approve ordinary deterministic reconciliation.

Human invocation for diagnostics is permitted. Human interpretation as a normal
dependency of correctness is not. Diagnostics MUST NOT be the mechanism by which
the product determines whether ordinary operations succeeded.

If semantic judgment would be required to establish or verify a requested state —
for example because the requested state is not mechanically observable — Lithify
MUST return that inability to its caller rather than pretending the problem is
mechanical.

## 0.12 No semantic orchestration

Lithify is not:

- an engineering project manager;
- an agent scheduler;
- an agent router;
- a backlog prioritizer;
- a workflow engine;
- a coding harness;
- a semantic task planner;
- an LLM-based project-management assistant;
- a replacement for Turnlock;
- a replacement for Ruu.

A higher-level system may use Lithify as its durable work-state boundary:

```text
policy / orchestrator / agent
            │
            │ explicit desired work state
            ▼
         Lithify
            │
            ▼
 durable work-state backend
```

For example, another system may decide:

```text
Issue 42 should become Ready
Issue 57 should block Issue 42
Issue 12 should be assigned to agent X
```

Lithify's responsibility begins once those desired states have been explicitly
selected. Normatively:

- Lithify MUST NOT choose which work exists, which item is prioritized, whether
  an item is ready, whether an item is complete, which agent receives a job,
  whether one work item depends on another, whether a finding is valid, or what
  workflow executes next;
- Lithify MUST NOT infer a desired state from observation: observing that an item
  is unassigned is not a request to assign it, and observing that two items look
  related is not a request to link them;
- every mutation Lithify performs MUST be attributable to an explicit desired
  state supplied by a caller; observation alone, however complete, is not
  mutation authority;
- when a request is under-specified, semantically ambiguous, or requires a
  judgment Lithify is not authorized to make, the conforming behavior is to
  return that fact to the caller, not to select an interpretation.

## 0.13 Relationship to workflow execution and version control

Lithify's state domain is engineering work state. Version-control state is a
different domain.

A system such as Ruu may own Git convergence, while a system such as Turnlock may
own workflow execution and orchestration. A larger autonomous engineering system
may compose them:

```text
durable work state ── Lithify
workflow execution ── Turnlock
Git convergence    ── Ruu
```

These products may integrate at a higher layer without transferring their
respective authority boundaries into Lithify. Lithify MUST NOT acquire
dependencies on Turnlock or Ruu as part of its product semantics, and their
mechanisms MUST NOT appear in Lithify's product contract.

A requested engineering-work state may legitimately reference an external
artifact by identity — for example, the linkage of a work item to an execution
artifact or a change artifact. Establishing and verifying that linkage is durable
engineering work state and is inside Lithify's domain. Establishing or verifying
the artifact's own internal state, its version-control convergence, or its
workflow progression is not.

## 0.14 Product-intent conformance rule

A proposed architecture or implementation is not product-conformant if ordinary
operation requires any of the following:

```text
a human to determine whether a mutation succeeded
a human to disambiguate which external object an operation targets
a human to inspect the backend to confirm ordinary success
a human to repair routine backend representation differences
a caller to translate every operation into provider-specific mechanics
a human to approve ordinary deterministic reconciliation
success to be reported because a backend accepted a mutation request
success to be reported without a fresh post-mutation observation of the requested state
an ambiguous target to become mutation authority by heuristic selection
an unverified outcome to be reported as certain success
an unobservable postcondition to be reported as a mechanical success
caller-owned policy (existence, priority, readiness, assignment, dependency, workflow choice) to be decided by Lithify
a backend-specific identifier, API shape, or provider object model to define Lithify's product semantics
a required semantic judgment to be silently converted into a mechanical-looking result
```

Fail-closed behavior remains required for unavailable, ambiguous, or
unverifiable state. Fail-closed behavior MUST NOT be used as a substitute for
mechanical actuation that the requested state requires and the backend permits.

## 0.15 Reference product-conformance scenario

The reference scenario is an unattended caller establishing a durable engineering
work state:

```text
1. An autonomous caller determines that an engineering-work state should exist.
   That decision is made outside Lithify.

2. The caller requests that state from Lithify, identifying the external work
   item and the desired state.

3. Lithify resolves the exact external target from current backend reality.

4. Lithify observes the current durable state of that exact target.

5. Case A — the requested state is already observed to hold:
     Lithify reports "already true" without mutating.
   Case B — the requested state is not observed to hold:
     Lithify performs only the mechanical mutation required and then
     re-observes the target.
   Case C — the target cannot be resolved exactly, or the observation cannot
     establish the state:
     Lithify reports "not proven" and names what could not be established.

6. In case B, the postcondition is established by a fresh observation that
   follows the mutation. The mutation's own response is not evidence.

7. The caller receives a machine-readable result distinguishing already true,
   made true and verified, and not proven.

8. The caller may retry, resume after a crash, or run concurrently with other
   callers. A repeated request for a state that is already observed to hold is
   again "already true", not a distinct effect.

9. No human interpretation is required at any step.
```

Two additional scenarios fix failure behavior.

```text
A caller requests a state change.
Lithify resolves the target and observes that the change is required.
The mutation is attempted and the backend acknowledges it.
A subsequent observation cannot be taken, or does not show the requested state.

The conforming outcome is a not-proven result that does not manufacture
confidence in either direction.
```

```text
Two external objects plausibly match the caller's target description.

The conforming outcome is a not-proven result that names the ambiguity.
The caller — not a human operator — decides what to do about it.
```

For any candidate architecture, it must be possible to answer, per operation:

```text
which exact external target was resolved, and from which observation
what durable state was observed before actuation
what mechanical mutation, if any, was required
what fresh observation established, or failed to establish, the postcondition
what machine-readable outcome was reported, and what remains uncertain
```

An architecture that cannot answer these questions for its ordinary operations has
drifted away from Lithify's product intent, regardless of how reliable its
plumbing is.

# 1. Specification discipline

This document follows the derivation discipline used by the Ruu and Turnlock
specifications. The following rules govern its own evolution and every artifact
that derives from it:

```text
1. Product Intent is normative.
2. Lower-level architecture must derive from it.
3. Implementation convenience cannot silently weaken the Product Intent.
4. Mechanisms remain unspecified until the product contract requires them.
5. Backend-specific accidents must not become fundamental semantics.
6. Unknown product decisions remain explicitly unresolved rather than being guessed.
7. Future product-semantic changes require an explicit recorded decision rather
   than appearing silently through implementation.
```

Consequences of these rules:

- This document MUST NOT be made to look complete by inventing lower-level
  design. Architecture, protocols, storage, process model, API syntax, concrete
  schemas, and language choices are absent because they are not yet required by
  the product contract, not because they were overlooked.
- Section 0 governs. Any later section that conflicts with Section 0 is wrong and
  MUST be corrected or removed; a conflicting intent claim cannot be introduced
  by adding more text below.
- An unresolved question recorded in Section 3 MUST NOT be treated as settled
  because an implementation happens to answer it. An implementation choice is
  evidence for a decision, never a decision.

# 2. Origins and implementation evidence (non-normative)

This section is context. It creates no requirement, promise, or invariant, and it
MUST NOT be used to interpret Section 0 more narrowly or more broadly than
written.

## 2.1 How the problem was discovered

- Repository-local `backlog.md` files were initially replaced with GitHub
  Issues and GitHub Projects.
- Agents then began creating Issues that represent future work, which made the
  backlog itself a shared durable artifact rather than a local file.
- GitHub consequently became durable work state and a coordination substrate for
  future agent execution.
- Existing ticketing abstractions already solve much of the multi-backend
  ticket-management problem. Issue and project systems, and the layers around
  them, are mature.
- The missing primitive identified here is therefore not another ticket manager
  or project-management layer.
- The narrower problem is reliable unattended actuation of durable work state,
  and proof that the requested state was actually reached.

## 2.2 Current implementation context

The repository `fanilosendrison/proto-lithify` is useful evidence
for the initial GitHub realization. It already demonstrates some of the
mechanical ideas this specification requires:

- explicit operations;
- live backend identity resolution;
- fail-closed ambiguity handling;
- mutation followed by a fresh read;
- postcondition verification;
- machine-readable receipts.

That repository is not migrated, copied, or refactored as part of this
specification work, and it is not an architectural template for Lithify. Not
every current implementation choice in it is part of Lithify's Product Intent.
It is evidence that the mechanics described in Section 0 are realizable against
GitHub; Section 0 alone decides what Lithify is.

# 3. Current boundaries — intentionally not specified

The following questions are material to the product but are **not decided** by
this specification. They MUST remain unresolved until an explicit product
decision resolves them; an implementation MUST NOT silently settle them.

- The exact common semantic model of durable engineering work state, including
  which concepts are first-class and how they compose.
- Whether Lithify owns any continuous, scheduled, or repeated reconciliation loop
  over previously requested state, or remains strictly request-scoped.
- Concurrency semantics between simultaneous operations on the same or
  overlapping targets, including whether coalescing, serialization, or any
  claim/lease mechanism is a product requirement.
- The taxonomy of non-success outcomes: whether "postcondition observed not to
  hold" and "postcondition unobservable / mutation outcome unknown" must be
  distinct product-level classes, or whether a single not-proven class suffices.
- How asynchronous or eventually materialized backend state is treated, including
  whether a still-materializing operation is a legitimate product-level outcome
  or whether Lithify must obtain an observation before returning.
- What counts as an admissible independent observation for a backend whose only
  post-write read is not distinguishable from the write acknowledgement.
- What constitutes an authorized caller, and whether authorization is Lithify's
  concern or the surrounding environment's.
- Whether resolved backend identities may be cached or carried between
  operations, and what freshness those identities must have.
- The result/receipt schema, its versioning, and whether operation identity must
  be durable and caller-visible.
- Whether multi-object or transactional requests are product operations, and what
  partial-effect reporting is required if they are.
- Whether Lithify is an invocation-scoped command, a long-running service, a
  library, or several of these, and whether any human-facing surface is part of
  the product or only an implementation convenience.
- Whether the first GitHub realization's operation set is the initial product
  surface or only evidence for it.

These are future derivation points. Their solutions must preserve Section 0.
