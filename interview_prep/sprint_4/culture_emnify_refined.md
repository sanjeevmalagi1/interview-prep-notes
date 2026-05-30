# Culture Interview @ Emnify

Talking points:

## Everyone is an architect
### Principle
Teams that code the system design the system.  
Decentralised ownership, not a separate architecture function.

### Example
- **Aspire**  
  Architected PAR (Product Activation Request).  
  - Started with a single feature  
  - Extended into a reusable framework for multiple product flows  
  - Enabled other engineers to build new PARs using consistent patterns  

- **Invense**  
  As a part-time contractor, owned both design and implementation.  
  - Acted as both architect and engineer  
  - Delivered production-ready systems end-to-end  

---

## Limit complexity
### Principle 
Build the simplest architecture that can possibly work.

### Example
- **Invense**  
  - Started with a lean serverless Node.js architecture  
  - Scaled to ECS only when client load and cost justified it  
  - Avoided premature complexity  

---

## When in doubt, code it out — explore alternatives
### Principle 
Proof-of-concept over speculation.

### Example
- **Scripbox**  
  - Evaluated communication patterns via POCs  
  - Tested Event-Driven Architecture and Outbox pattern  
  - Chose Outbox (via Debezium) due to team alignment and PostgreSQL usage  

- **GenAI / Observability**  
  - Built PoC using Jaeger for distributed tracing  
  - Used it to reduce debugging time and improve visibility  

---

## They build it, they test it — fast learning cycles
### Principle
Ownership end-to-end.

### Example
- **Aspire & Scripbox**  
  - Owned systems from development to production  
  - Responsible for technical outcomes  
  - Continuously improved systems based on feedback  

---

## Keep variability — late decisions, Set-Based Design
### Principle
Leave options open, don’t foreclose early.

### Example
- **Invense**  
  - Designed IoT platform supporting energy, fuel, and solar  
  - Same architecture reused across multiple verticals  

- **Aspire**  
  - Integrated external providers (Feathery, Jotform)  
  - Built domain-driven abstractions to avoid vendor lock-in  

---

# Scale Pattern

## Event-driven architecture
### Principle 
Core pattern — messaging, async pipelines.

### Example
- **Aspire & Scripbox**  
  - Used Outbox pattern with Debezium  
  - Leveraged PostgreSQL as source of truth  
  - No dedicated producers — DB writes triggered events  

---

## Microservices with API standards
### Principle
Decomposed, independently deployable units.

### Example
- **Scripbox**  
  - Migrated to microservices as complexity increased  
  - Standardised APIs using OpenAPI  
  - Ensured consistency across services  

---

## Scalability & resiliency
### Principle
System must hold under load and failure.

### Example
- Use tools like autocannon for load testing  
- Building a framework to integrate performance testing into CI/CD  

---

## Infrastructure & cloud services layer
### Principle
Deliberate infra choices, not just managed services.

### Example
- **Invense**  
  - Started with serverless for MVP  
  - Identified workload patterns:
    - High-throughput ingestion → moved to ECS  
    - Variable user traffic → kept serverless  
  - Balanced cost, performance, and scalability  

---

## Platform evolution — domain model & transitional steps
### Principle
MVP → target architecture with clear steps.

### Example 
- Always design with future evolution in mind  
- Define next steps early to avoid rewrites  
- Incrementally evolve architecture as scope grows  

---

# @Scale culture - people

## Architects are leaders — coach & teach, listen & learn

### Example
- **Aspire**  
  - Built PAR as a framework, not just a feature  
  - Used strategy pattern for extensibility  
  - Documented design for team-wide adoption  

---

## Shared ownership — platform arch, shared services

### Example
- **Aspire**  
  - Built A/B testing framework used across teams  
  - Teams owned experiment logic  
  - Core platform owned centrally for consistency  

---

## Domain ownership & divide and conquer

### Example
- **Aspire**  
  - Owned PAR and A/B testing frameworks  
  - Responsible for long-term system health  

- **Scripbox**  
  - Owned financial data aggregation system  
  - Exposed APIs and Kafka events for other teams  

---

## Trust must be earned — handle with care

### Example 
- **Scripbox**  
  - Transitioned from frontend to backend  
  - Built trust by fixing backend issues and contributing incrementally  

- **Aspire**  
  - Building trust by delivering shared platform features  
  - Adoption by teams increased confidence  

---

## Communication — synchronous preferred, async by design

### Example
- Remote collaboration across geographies  
- Daily syncs + strong async documentation (tech specs)  
- Delivered effectively without co-location  

---

# Agile Architecture @ Org Scale

## Feature teams + system architects working together

### Principle
Architects embedded, not siloed.

### Example
- **Invense**  
  - Defined system architecture while internal teams built features  

- **Aspire**  
  - Worked across onboarding, KYC/KYB, payments, multi-currency  
  - Understand cross-team dependencies  

---

## Technology strategy — define tech, frameworks, and practices

### Example
- Choose tech based on requirements, not trends  
- Validate decisions with POCs  
- Focus on utility and long-term maintainability  

---

## Architecture reviews — intentions upfront, body of knowledge

### Example
- **Aspire**  
  - Write detailed tech specs before implementation  
  - Clearly define:
    - Current system  
    - Proposed system  
    - Problems being solved  
  - Peer reviews ensure alignment and knowledge sharing  

---

## Cross-team alignment under a release cadence

### Example
- **Aspire**  
  - Follow strict 2-week release cycles  
  - Feature freeze and coordinated testing (FCT)  
  - Teams align deliverables with release cadence  

---

# Architecture Review

## Lightweight docs — business problem, goal (done), multiple lenses
### Example
- Keep documentation concise but structured:
  - Define business problem clearly  
  - Define "done" criteria upfront  
  - Avoid over-documentation, focus on clarity  
- Ensures faster reviews and better alignment across teams  

---

## Different lenses — business process, application, implementation, deployment
### Example
- Break down system design into multiple perspectives:
  - Business flow (user journey, process)  
  - Application interactions (service communication)  
  - Implementation (APIs, data models)  
  - Deployment (infra, scaling)  
- Helps stakeholders at different levels understand the system  

---

## Focus on exceptions — clarify scope, invoke discussions
### Example
- Highlight edge cases and ambiguities early:
  - Failure scenarios  
  - Data inconsistencies  
  - External system dependencies  
- Drives meaningful discussions instead of superficial approvals  

---

## ARB — visibility and alignment across all stakeholders
### Example
- Architecture Review Board ensures:
  - Cross-team visibility  
  - Alignment on major decisions  
  - Avoidance of conflicting designs  
- Helps scale architecture decisions across the organization  


# Discussions about culture

- I know you have joined Emnify recently. Why did you join?
- What were some of your expectations you had which came true?
- What are improvements you plan on bringing? if any.
- How do you evaluate people, especially engineers? How do you evaluate teams? Do you have any framework?
- How do you reward people who go above and beyond in their role?
- How do you work with people who need improvements?