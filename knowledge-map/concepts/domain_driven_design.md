# Domain Driven Design

## Overview

Domain Driven Design (DDD) is an approach to software design, introduced by
Eric Evans (*Domain-Driven Design*, 2003), that puts the **business domain**
— not the database, not the framework, not the UI — at the center of the
software's structure. The core idea: talk to domain experts, build a shared
vocabulary with them (the **ubiquitous language**), and shape your code's
types, methods, and module boundaries around that vocabulary so the code
reads like the business, not like plumbing.

**Rule of thumb:** reach for DDD when the domain itself is complex (lots of
business rules, invariants, and edge cases that change independently of
infrastructure). It's overkill for CRUD-shaped apps where the "domain logic"
really is just create/read/update/delete against a database.

---

## Core building blocks

### Ubiquitous language

A shared vocabulary between engineers and domain experts (product, support,
ops) that's used consistently in conversation, docs, *and* code — the class
named `Submission` in code should be called "submission," not "record" or
"entry," in every meeting and ticket too. Mismatched vocabulary (code says
`FormResponse`, PMs say "submission") is a signal the model has drifted from
the domain.

### Entities

Objects with a distinct, persistent **identity** that survives changes to
their attributes over time — two entities with identical field values are
still different objects if their IDs differ, and the same entity is still
"the same thing" after its fields change. Example: a `Submission` with id
`sub_123` is the same submission whether it's `status=draft` or
`status=completed`.

### Value objects

Objects defined entirely by their **attributes**, with no identity — two
value objects with the same values are interchangeable and (ideally)
immutable. Example: a `Money(amount, currency)` or a `FieldAnswer(fieldId,
value)`. You don't ask "which one" — you only ask "what value."

### Aggregates

A cluster of entities and value objects treated as a single consistency
boundary, with one entity as the **aggregate root** — all external access
goes through the root, which enforces the invariants for everything inside
it. Example: a `Form` aggregate might contain `FormField` value objects; you
never mutate a field directly, you go through `form.updateField(...)` so the
form can enforce "at least one required field" or "field IDs are unique."
Aggregates are also the natural transaction/consistency boundary — one
aggregate = one thing you can atomically save.

### Repositories

An abstraction that looks like an in-memory collection but is backed by
persistence — `FormRepository.findById(id)`, `SubmissionRepository.save(s)`.
The point is that domain code talks to repositories, not to SQL or an ORM
directly, so persistence details don't leak into business logic.

### Domain events

Something that happened in the domain that other parts of the system care
about — `SubmissionReceived`, `FormPublished`. Modeling these explicitly
(rather than just mutating state) makes side effects (notify a Slack
channel, kick off a workflow) explicit and decoupled, and pairs naturally
with event-driven architecture.

### Bounded contexts

The most important strategic (as opposed to tactical) DDD concept. A large
system usually has multiple, only loosely related, notions of the "same"
concept — "customer" means something different to Billing than it does to
Support. A **bounded context** is an explicit boundary (often = a service or
module) within which a model and its ubiquitous language are consistent —
outside that boundary, the same word can mean something else, and that's
fine as long as the boundary is explicit. Contexts talk to each other
through defined integration points, not by sharing an internal model.

### Anti-corruption layer (ACL)

A translation layer at the boundary of a bounded context that converts an
external system's model into your domain's model, so the external system's
quirks, naming, and shape don't leak in and "corrupt" your domain model.
This is the piece that matters most for the example below.

---

## Worked example: form-builder integrations

> Your example: at your org, external form builders (Typeform, JotForm,
> SurveyMonkey) are used to collect information from users, and internally
> the domain is modeled around `Form` and `Submission` — so a new provider
> can be onboarded without the rest of the system changing.

**Yes — this is a solid, textbook-appropriate DDD example.** It cleanly
illustrates several of the concepts above at once, which is worth calling
out explicitly if you use it:

- **Bounded context + anti-corruption layer** — this is the strongest part
  of the example. Each provider has its own API shape (Typeform calls it a
  "response," JotForm has its own field-type enum, SurveyMonkey nests things
  differently). Your integration layer for each provider is an ACL: it
  translates *their* model into *your* domain's `Form` / `Submission`
  vocabulary. The rest of the system never sees a Typeform response or a
  JotForm payload — it only ever sees your own `Submission` entity.
- **Ubiquitous language** — `Form` and `Submission` are stable,
  provider-agnostic nouns that your team, PMs, and code all use the same
  way, regardless of which external tool actually collected the data.
- **Entities and aggregates** — `Form` (aggregate root, with `FormField`s)
  and `Submission` (aggregate root, with `Answer` value objects) are a
  natural fit; each submission has identity independent of its field values
  changing (e.g., going from `partial` to `completed`).
- **Open-Host Service / adapter pattern (supporting piece)** — the "new
  provider, same underlying concepts" scalability you describe is really
  what happens when the ACL is implemented as one adapter per provider
  behind a common interface, e.g. `FormProviderAdapter.normalize(rawPayload)
  -> Submission`. DDD doesn't mandate a specific adapter pattern, but the
  ACL concept is exactly *why* this is easy: onboarding provider #4 means
  writing one new adapter, not touching the domain model or anything
  downstream of it (validation, storage, notifications, analytics).

A sketch of the boundary:

```
Typeform payload ──┐
JotForm payload ────┼──> [Adapter per provider = ACL] ──> Submission (domain entity)
SurveyMonkey payload┘                                            │
                                                                  v
                                            downstream domain logic (storage,
                                            validation, notifications, analytics)
                                            — written once, against Submission,
                                            never against a provider's shape
```

**One thing to be precise about when you present this**: the payoff isn't
just "one model for many providers" (that's arguably just data
normalization) — the DDD framing is specifically that the *domain* concepts
(`Form`, `Submission`, their invariants and behavior) are deliberately kept
free of any provider's vocabulary or shape, and the translation is isolated
at the edge (the ACL). That's the part that makes it a DDD example rather
than just "we wrote a normalizer."

---

## Strategic vs tactical DDD

- **Strategic DDD** — the big-picture decisions: identifying bounded
  contexts, mapping how they relate (**context mapping**: e.g.
  customer/supplier, conformist, anti-corruption layer, shared kernel), and
  deciding which context is the core domain worth investing the most design
  effort in vs. which are generic/supporting subdomains.
- **Tactical DDD** — the building blocks above (entities, value objects,
  aggregates, repositories, domain events) used *within* one bounded
  context to model that context well.

Most teams get more value from strategic DDD (getting the boundaries right)
than from applying every tactical pattern everywhere — over-applying
aggregates/repositories/entities to a simple CRUD context is a common
over-engineering trap.

---

## When DDD is worth it (and when it isn't)

**Worth it:**
- The domain has real complexity — many business rules, invariants that
  must hold across multiple fields/entities, behavior that changes based on
  domain-expert input rather than technical constraints.
- Multiple teams/services need explicit boundaries to avoid a shared,
  tangled model (this is where bounded contexts pay off most).
- You're integrating with external systems whose models shouldn't leak into
  yours (the form-builder example above).

**Not worth it:**
- Simple CRUD apps where the "domain logic" is just persistence with
  validation.
- Small, single-team codebases where the overhead of aggregates/repositories
  exceeds the complexity being managed.
