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

## 0.1 Product definition: a reliable mechanical boundary for durable engineering work state and records

Lithify is a **reliable, backend-independent mechanical boundary for durable engineering work state and engineering work records**. It enables autonomous callers to establish explicitly requested work state, publish and retrieve explicitly supplied work records, and verify the relevant backend facts without relying on transient agent context or human interpretation.

Lithify exists to give autonomous systems a **reliable, backend-independent
mechanical boundary** for three things:

```text
establishing explicitly requested durable engineering work state
establishing and retrieving explicitly supplied durable engineering work records
proving requested state, publication, or retrieval postconditions from actual backend observations
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

Publication and retrieval are also request-scoped mechanical operations:

```text
explicit record request → exact destination resolution → observation → required publication → fresh observation → verified outcome
explicit retrieval request → exact scope/identity resolution → backend observation → content and relationship verification → verified result or not-proven outcome
```

A read-only retrieval does not require mutation. These sequences establish product
boundaries, not an internal process model, protocol, or receipt schema.

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

An autonomous caller MUST also be able to supply a work record for publication,
retrieve an existing record, and obtain backend evidence of the requested result.
Lithify MUST resolve the exact destination, identity, or work scope and verify
publication or retrieval postconditions. A publication mutation requires fresh
post-mutation observation, not merely backend acknowledgement. For retrieval,
success requires observed content and relevant associations under the declared
retrieval contract, not a mutation.

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
- what workflow should execute next;
- whether a record should be published and which information is submitted;
- which records should be selected and which work item or scope they concern;
- which relationships and authorized backend destination are requested;
- whether a record supersedes or corrects an earlier record;
- how a record is interpreted or influences subsequent execution.

Lithify MUST NOT decide whether a reported `DONE`, `STOP`, or `PARTIAL` means a
work item should change state. Publication MUST be explicitly requested and
authorized; Lithify MUST NOT automatically capture full agent contexts, private
conversations, secrets, or unrelated data.

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

## 0.5 The product domain: durable engineering work state and records

The product domain comprises durable engineering work state and explicitly
supplied engineering work records. It may include concepts
such as:

- durable work items;
- workflow state;
- classification;
- priority;
- assignment;
- dependencies / blockers;
- parent-child work relationships;
- linkage to execution or change artifacts;
- other durable state required by engineering-work systems;
- received instructions, task packets, and caller-supplied assignment context;
- execution reports, including completion, STOP, PARTIAL, blocked, or uncertain reports;
- explicitly submitted findings, observations, and authorized caller decisions;
- explicit relationships between instructions, executions, reports, decisions,
  subsequent work, and work items.

Publishing a record's contents is distinct from linking an arbitrary external
artifact. Record existence, correct identity, content fidelity, provenance,
work relationships, availability at observation time, and long-term retention or
immutability MUST remain distinct claims. Lithify MUST NOT claim more than the
backend evidence establishes. It verifies the supplied record's publication and
retrieval, not the truth of its statements, execution correctness, or semantic
validity of a `DONE` result.

This list MUST NOT be frozen into an implementation-specific universal schema.
Necessary product obligations MUST be derived from the accepted Product Intent,
not from existing backend objects, prototype APIs, or observed work practices.
Concrete backend evidence informs the evaluation of candidate representations
and realization choices; it does not establish normative necessity. Product
choices not entailed by accepted premises require explicit product authority.
No concept becomes a required Lithify primitive merely because GitHub,
Proto-Lithify, or an existing workflow exposes or uses it.

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
backend-independent work-state or work-record operation
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

This section does not authorize a speculative universal backend abstraction.
GitHub remains the first selected backend, but its object model MUST NOT supply
premises for deriving Lithify's necessary product obligations. A realization
that currently supports only GitHub may conform to the backend-independent
product boundary.

A shared backend abstraction may be considered as a realization candidate.
Claims about what is common across backends MUST be supported by evidence from
at least two concrete backend realizations. This evidence condition governs
empirical generalization of backend mechanisms; it is not a prerequisite for
independently deriving product obligations from accepted Product Intent, and
it does not itself authorize promotion of an abstraction into product semantics.

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

Publication and retrieval are subject to these same observed-postcondition
requirements. Successful publication MUST establish the requested content
representation, durable record identity, and requested work association.
Successful retrieval MUST identify the retrieved record and verify its represented
content and relevant associations against the declared retrieval contract.
Authorized callers MUST be able to discover records associated with an explicitly
identified work item or equivalent scope and establish and verify explicitly
requested relationships between records and work items. Incomplete, ambiguous,
inaccessible, or unverifiable results MUST be reported rather than hidden.

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

For publication, the same fresh post-mutation observation is required. Read-only
retrieval MUST use backend observation to establish its requested postconditions;
it does not require a write or a post-write observation.

## 0.9 Exact identity and fail-closed ambiguity

Autonomous unattended mutation, publication, and retrieval require exact targeting. Lithify MUST NOT rely on
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

For a new record, the containing destination MUST be resolved before publication;
the created record's final identity MAY be established and verified afterward.
No pre-creation backend identity is required for a nonexistent record. Retrieval
MUST resolve the requested identity or explicitly identified work scope without
heuristically selecting among ambiguous records.

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

These guarantees also apply to record publication, relationship establishment,
and retrieval. Successful publication does not establish permanent immutability,
future availability, or exactly-once effects. Uncertain publication, duplicate
risk, and concurrent changes MUST NOT be hidden by a success claim.

This section does not fix a concurrency-control, serialization, coalescing, or
claim model (Section 3).

## 0.11 Machine-first operation and human surfaces

Lithify MAY expose human-accessible diagnostics, inspection commands, debugging
tools, or administrative interfaces. Those are secondary surfaces.

A conforming runtime MUST NOT require a human to:

- interpret whether a mutation, record publication, or retrieval succeeded;
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

- an agent memory model;
- an automatic conversation recorder;
- a semantic report interpreter;
- a reasoning engine;
- an independent execution or review verifier;
- a replacement for the external durable backend;
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

A higher-level system may use Lithify as its durable work-state and record boundary:

```text
policy / orchestrator / agent
            │
            │ explicit desired work state or record publication/retrieval request
            ▼
         Lithify
            │
            ▼
 durable engineering-work backend
```

For example, another system may decide:

```text
Issue 42 should become Ready
Issue 57 should block Issue 42
Issue 12 should be assigned to agent X
publish this supplied instruction for the identified work
publish this supplied execution report and its requested relation to that instruction
retrieve the available records associated with the identified work
```

Lithify's responsibility begins once those desired states have been explicitly
selected, or publication or retrieval has been explicitly requested. Lithify MUST
NOT automatically interpret agent execution reports or infer continuation from
them. Normatively:

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

Lithify's domain is engineering work state and records. Version-control state is
a different domain.

A system such as Ruu may own Git convergence, while a system such as Turnlock may
own workflow execution and orchestration. A larger autonomous engineering system
may compose them:

```text
durable work state and records ── Lithify
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

Lithify MAY establish or retrieve caller-supplied reports and their explicit
relationships, including references to execution artifacts, without assuming
responsibility for execution correctness. External systems execute workflows,
produce reports, review results, and converge version-control state. Verifying a
record's represented content under its retrieval contract is not independently
verifying the referenced execution or artifact's internal correctness. No such
system, including Review Broker, becomes a normative dependency of Lithify.

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
a record to be reported as published solely because a write request was accepted
content to be claimed durably preserved when only an unverified external pointer was stored
the originating agent's transient context to be required for ordinary retrieval
records to be silently selected by heuristic while identity or scope remains ambiguous
semantic validity or task completion to be inferred from a published report
indefinite retention or immutability to be claimed without an established guarantee
unauthorized automatic capture or publication
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

A second product-conformance scenario establishes cross-agent continuity:

```text
1. An authorized caller explicitly submits an instruction for publication.
2. Lithify resolves its destination, establishes its durable record identity,
   and verifies publication through fresh backend observation.
3. An execution report is explicitly submitted for publication, with a requested
   relationship to the instruction and identified engineering work.
4. Lithify establishes and verifies the report and requested relationships.
5. The originating agent terminates or loses its transient context.
6. A different authorized agent identifies the engineering work and discovers
   and retrieves the available instruction and report through that work.
7. Backend observations establish returned content, record identities, and
   relationships without the originating agent's transient context.
8. The receiving agent or orchestrator decides the next action outside Lithify.
```

If the backend accepts publication but Lithify cannot verify the record's content
or required association, the conforming outcome is not-proven, not success.

For any candidate architecture, it must be possible to answer, per operation:

```text
which exact external target was resolved, and from which observation
what durable state or record evidence was observed before any required actuation
what mechanical mutation, if any, was required
what fresh observation established, or failed to establish, the postcondition
what machine-readable outcome was reported, and what remains uncertain
```

An architecture that cannot answer these questions for its ordinary operations has
drifted away from Lithify's product intent, regardless of how reliable its
plumbing is.

## 0.16 Durable agent work records and cross-execution continuity

Lithify MUST provide authorized autonomous callers with mechanical capabilities to establish, retrieve, and verify durable engineering work records associated with explicitly identified engineering work.

A work record represents caller-supplied engineering-work information intended to remain accessible outside the originating execution context. Such information MAY include an instruction or work packet received by an agent, an execution report produced by an agent, an observation, a finding, a decision, or a relationship to another work record or work item. These examples do not prescribe a universal record schema.

The product-defining outcome is that an agent, or another authorized agent executing later, can recover the established work records through durable backend identities and work associations without relying on the originating agent's transient memory, conversation state, process lifetime, or local execution context.

Lithify MUST NOT decide which information deserves publication, what its contents mean, what action follows from it, or whether a report's substantive claims are correct. Those decisions and judgments belong to the caller or another higher-level system.

For an explicitly requested record publication, Lithify MUST resolve the authorized destination, attempt only the required mechanical operation, and report verified success only when a fresh backend observation establishes the requested record identity, content representation, and required associations. A backend write acknowledgement is insufficient.

For an explicitly requested record retrieval, Lithify MUST resolve the requested identity or work scope, obtain backend evidence, and distinguish verified retrieval from absence, ambiguity, incompleteness, or inability to establish the requested content and associations.

Where a record's content is represented through an external reference, verification of that reference alone MUST NOT be reported as verification of the referenced content's durability or availability.

Where the backend cannot establish unique publication, content fidelity, retrieval, or requested relationships, Lithify MUST expose the relevant uncertainty rather than manufacture a successful result. Retry and concurrency behavior MUST preserve the existing fail-closed guarantees.

Lithify MUST NOT claim that publication success establishes indefinite retention, immutable historical preservation, global atomicity, exclusive causality, or future availability unless the relevant guarantee is separately established.

The external backend owns the stored records. Lithify owns the mechanical boundary through which their requested publication, retrieval, and relationships are established and verified.

The specific record vocabulary, interface, schema, indexing strategy, retention policy, revision model, and backend realization remain architectural or product decisions to be derived separately. This subsection establishes required product capabilities, not their implementation.

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

## 1.1 Recorded methodological decision — 2026-11-04

The product owner explicitly separates Lithify's independent product derivation
from Proto-Lithify's empirical investigation. Proto-Lithify is not a mandatory
architectural predecessor, a normative reference implementation, or a template
that Lithify must generalize.

This decision also clarifies Sections 0.5 and 0.6: backend evidence supports
realization evaluation and justified claims of cross-backend commonality; it
is not the source of Lithify's necessary product obligations. This is an
explicitly recorded owner decision, not a claim that empirical practice has
proved a new necessary product requirement.

## 1.2 Independent derivation and explicit choices

For each proposed obligation, identify its accepted premises and distinguish:

- a necessary consequence of accepted Product Intent;
- an explicitly accepted product choice that is not logically entailed;
- a candidate architectural or realization choice;
- an empirical observation or a remaining uncertainty.

Do not treat an existing implementation, backend API, GitHub object, command,
agent workflow, or work-record taxonomy as a normative premise merely because
it exists or appears useful. Compatibility with Product Intent does not prove
necessity. An unproven consequence remains unproven; unresolved choices must
not be filled by an implementation preference.

This discipline does not claim that Product Intent uniquely determines every
architecture. Where multiple realizations remain compatible with established
obligations, preserve that freedom and identify the separate decision needed
before selecting one.

## 1.3 Empirical challenge and later confrontation

Proto-Lithify's empirical work may proceed independently and in parallel.
Observations may expose limitations, counterexamples, or questions requiring
examination. They do not silently amend Product Intent or retroactively become
the premises of an earlier derivation.

Before using Proto-Lithify to select a Lithify realization, establish and accept
an independently derived obligation baseline for the exact scope being
confronted. Identify its revision, premises, unresolved decisions, exclusions,
and justification. This does not require all future Lithify derivation to be
complete before any bounded comparison can occur.

Confront that baseline with separately identified empirical evidence. Record
which obligations are supported, unsupported, contradicted, or unexamined by
the evidence; distinguish missing evidence from a demonstrated incompatibility.
Keep GitHub-specific constraints explicit and assess alternatives rather than
presupposing prototype reuse.

A confrontation may recommend reuse, adaptation, replacement, rejection, or
continued investigation. No outcome is predetermined. Adoption requires its
own justified decision under the accepted product boundary. Any necessary
change to Product Intent requires an explicit owner decision and synchronized
normative text, not a retrospective reinterpretation of experimental results.

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

## 2.2 Proto-Lithify as empirical evidence

The separate repository `fanilosendrison/proto-lithify` contains a concrete
GitHub adapter and is an empirical investigation ground for engineering work
state and work-record practices. Its existing implementation provides evidence
about explicit operations, live identity resolution, postcondition observation,
and machine-readable receipts within its declared contracts and tested scope.

Existing capabilities, proposed experiments, observations, qualification
results, and remaining limitations must be distinguished. A research role does
not establish that work-record publication or retrieval has been implemented,
and passing implementation tests does not establish complete live-provider or
Lithify product conformance.

Proto-Lithify's command vocabulary, GitHub object mappings, repository layout,
and observed agent workflows do not define Lithify's required concepts or
architecture. Its evidence is an input to the bounded confrontation described
in Section 1.3, not a normative source for independent derivation.

The prototype's empirical charter is maintained in its own repository at
`docs/research/empirical-role-and-evidence.md`. That charter governs how its
research is described and delimited; it has no authority to amend Lithify.
Lithify owns the assessment and any adoption decision concerning Lithify.

No migration, code reuse, generalized CLI, backend architecture, or common
work-record schema is selected by this relationship. Section 0 remains the
product authority; Section 3 retains the explicitly unresolved decisions.

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
- The concrete work-record semantic model.
- Whether record content is embedded, referenced, or both.
- Record identity, content representation, and integrity evidence.
- Discovery and association mechanisms for work-scoped records.
- Revision, correction, supersession, and historical-preservation semantics.
- Retention, deletion, access control, and confidentiality guarantees.
- Publication idempotence, concurrent publication, and recovery after uncertain outcomes.
- Backend support limitations for work records.
- The specific publication and retrieval API, receipt contracts, and indexing architecture.

These are future derivation points. Their solutions must preserve Section 0.
