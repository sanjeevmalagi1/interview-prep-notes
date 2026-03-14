Delivery Framework

Requirements -> Entities -> Class Design -> Implementation -> Extensibility


Requirements (~5 minutes)
- Primary capabilities
- Rules and completion
- Error handling
- Scope boundaries

Entities (~3 minutes)
- Identify Entities:
  - If something maintains changing state or enforces rules, it likely deserves to be its own entity.
  - If it’s just information attached to something else, it’s probably just a field on another class.

- Define Relationships:
  - Which entity is the orchestrator — the one driving the main workflow?
  - Which entities own durable state?
  - How do they depend on each other? (has-a, uses, contains)
  - Where should specific rules logically live?

Class Design (~10-15 minutes)
For each entity, you'll answer two questions:
  - State - What does this class need to remember to enforce the requirements?
  - Behavior - What does this class need to do, in terms of operations or queries?

- Deriving State from Requirements (properties)
  - Which parts of the requirements does this entity own?
  - What information does it need to keep in memory to satisfy those responsibilities?
- Deriving Behavior from Requirements (methods)
  - what operations the outside world needs.
  - which requirements those operations satisfy.

Implementation (~10 minutes)
- Happy Path
- Edge Cases

- Verification: Walk Through a Specific Scenario

Extensibility (~5 minutes)