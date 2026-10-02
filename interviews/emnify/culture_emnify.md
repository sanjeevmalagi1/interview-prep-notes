# Culture Interview @ Emnify

Talking points:

## Everyone is an architect
### Principle
Teams that code the system design the system.
Decentralised ownership, not a separate arch function.

### Example
01. At Aspire, I architected PAR (Product Activation Request). I designed the system and also I build it. 
First started for a single feature later extended it for other feature. Created a framework for other engineers to follow while creating other PARs.

02. At Invense, as a part-time contractor I designed the system, I build the system I was architect as well as engineer

## Limit complexity
### Principle 
Build the simplest architecture that can possibly work.

### Example
01. At Aspire, worked on 

02. At Invense, Kept the IoT architecture deliberately lean. Simple serverless architecture with NodeJS which worked until we had 2-4 clients then updated to use ECS as the client system requirements / costs went higher.

## When in doubt, code it out — explore alternatives
### Principle 
Proof-of-concept over speculation.

### Example
01. At Scripbox, while working deciding of communication between different systems we tested
- Event Driven Architecture, built POC for Outbox pattern & other pattern 

- Chose `Outbox` because all the teams supported the this.

02. Leveraging GenAI, built PoC for tracing (using jaeger) - spanning to test and use to reduce bugs.

## They build it, they test it — fast learning cycles
### Principle
Ownership end-to-end.

### Example
01. At both Aspire & Scripbox, all teams worked on ownership principle whatever I build, I owned it and owned technical outcomes and improved the system based on feedback and results.

## Keep variability — late decisions, Set-Based Design
### Principle
Leave options open, don't foreclose early.

### Example
At Invense,
The IoT system supported energy, fuel, and solar — multiple dashboard versions from the same platform. You designed for variability upfront so the same architecture served different customer verticals without rewrites.

At Aspire,
We used external form systems like Feathrey, Jotform but build the system to incorporate future changed based the system by the domain rather than the specifics of Jotform principles

# Scale Pattern
## Event-driven architecture
### Principle 
Core pattern — messaging, async pipelines.

### Example
At Aspire and Scripbox used both EVD.
for clear communication between teams used EVD using outbox pattern using Debezium. We used this because all of our systems had Postgress DB and using Debezium would prevent excessive integration. (No producers, just create entry into DB) 

## Microservices with API standards
### Principle
Decomposed, independently deployable units.

### Example
At Scripbox, as system went more complex we adapted Microservices architecture. 
Each microservice had.
- Standardised API patterns across all systems
- Standards enforced by OpenAPI

## Scalability & resiliency
### Principle
System must hold under load and failure.

### Example
I tend to use auto-cannon to test resilience of systems that I build.
I am currently working on framework to include this into CI/CD pipeline.

## Infrastructure & cloud services layer
### Principles
Deliberate infra choices, not just managed services

### Example
At Invense,
- I chose serverless architecture at beginning as MVP because it required no maintenance.
- But later as we onboarded more customers I figured out a pattern. The ingestion endpoints were always almost fixed / predictable usage but heigh usage. But the operational endpoint had custom usage (like when users login and use the system). So I used dedicated machines on AWS (ECS) for ingestion. Kept Serverless endpoints for Operational needs.

## Platform evolution — domain model & transitional steps
### Principle
MVP → target architecture with clear steps.

### Example 

I always think ahead of system. Not just whats working now. But how to improve it if the scope improves so that I would have a clear principles to begin with.

# @Scale culture - people

## Architects are leaders — coach & teach, listen & learn

### Example
At Aspire, while building PAR. Instead of just building the feature. I created a framework.
- It helped to expand the product activation request to work for all kinds of needs.
- The principled approach solved this. The documentation module design (strategy pattern) helped me achieve this.

## Shared ownership — platform arch, shared services

### Example
At Aspire, I build features like A/B testing framework and Product Activation Request.
- A/B testing framework helped the whole organization to test out their features and give the results.
Each the design of AB tests and outcome is owned by teams that use it. But the design of A/B testing framework is owned by me.

## Domain ownership & divide and conquer

### Example
At Aspire, I owned PAR product activation request framework. I owned A/B testing framework. I owned technical outcomes of these features. I make sure all these features and working as expected and will stay catered to user's needs.

At Scripbox, I owned Financial data aggregation system which owned that part of the business and exposed APIs and kafka events.

## Trust must be earned — handle with care

### Example 
At Scripbox, I started as front end engineer. But later transitioned to be a backend engineer.
To transition I needed to prove I could be trusted with responsibilities, so I started filling in gaps in backend (fixing bugs/updating APIs). Once I became confident and my colleagues became confident. I asked my CTO to transition me to a backend role.

Same at Aspire, Here I am in the process of building the trust. By releasing common features first and after people have started using my features I am earning the confidence.

## Communication — synchronous preferred, async by design

### Example
- Remote SDE3 at Aspire from Germany (Singapore HQ).

- Part-time advisor at Invense — you've operated without colocation repeatedly and still delivered. 

- Daily stand ups, but also tech specs.

# Agile Architecture @ Org Scale

## Feature teams + system architects working together

### Principle
Architects embedded, not siloed.

### Example
- At Invense you were the system architect while the internal team built features — you set the rails, they ran on them. 

- At Aspire you've shipped across onboarding, KYC/KYB, payments, and multi-currency — you understand cross-team surface area from both sides.

## Technology strategy — define tech, frameworks, and practices

### Example
- I chose tech based on requirements. 
- I always expriment with all required tech with POCs first. and test it and chose the right kind.
- Descisions should always based on utility

## Architecture reviews — intentions upfront, body of knowledge

### Example
- At Aspire, we all write tech specs, review each other's tech specs. In the specs we need the intentions upfront. we define current system and newer system and list out the problems to solve.

## Cross-team alignment under a release cadence

### Example
- At Aspire, all the teams follow a strict 2 week release cycle with feature freeze and FCT in place.
we all plan and align our stuff according to release cycle.

# Architecture Review

## Lightweight docs — business problem, goal (done), multiple lenses

## Different lenses — business process, application collaboration, implementation, deployment

## Focus on exceptions — clarify scope, invoke discussions

## ARB — visibility and alignment across all stakeholders



