# Answering framework
Context -> Decision -> Tradeoff -> Outcome -> Learning


## Tell me about yourself
- Born and bought up in a small town in India 
- Studied Instrumentation Technology
- Learnt Web development as a Hobby
- Joined a startup as a early-engineer which built IoT based parking-lot management company
- Built Backend, Dashboards and User Apps etc
- Later joined a Fintech named Scripbox as an early engineer as well
- Worked there as a both front end engineer and backend engineer
- First 3 years as frontend engineer, later 4 years as backend engineer
- Built Investment Platform (B2C customer facing application) in Web and App. (React and React Native)
- As backend engineer handled external accounts aggregation system. (Single source of truth for all external financial data of users)
- Also build financial instruments aggregations systems. (Single source of truth for all financial instruments like Banks, FD, Stocks, Stock Prices, Mutual Funds etc)
- Interacted with stack holders, Product Managers and Frontend Engineers. (Also in some cases external service providers)
- Exited Scripbox after 7 years after IPO.
- Currently working a backend engineer at Aspire, Fintech based in Singapore. Operated in Singapore, Hong Kong, US expanding operations in Australia and Netherlands (EU)
- I work on onboarding team. I handle all non-KYC/KYB compliance related projects
- I build systems which help improve interaction between our users and customers, while decreasing Cost of Acquisition
- Recently I build Product Activation Framework, A/B testing Framework and External Form Providers system
- Apart from my work, In free time, I am into fitness I go to gym everyday. I play video games. I build stuff (technical stuff). I do Origami (micro origami - my specialization - with tweezers and stuff).
- I also worked as part-time, Technical Advisor for IoT based telematics company.

## Why this company?
- Drive Transformation
“I enjoy building scalable systems and improving existing architectures. I’ve worked on moving systems toward more event-driven and flexible designs, which aligns with driving transformation.”

- Enable Customers
“I focus on solving real user problems and building reliable systems. Especially in fintech, I’ve seen how important it is that what we build directly improves customer experience.”

- Empower People
“I like taking end-to-end ownership of what I build — from design to production — and working collaboratively with teams. I do my best work in environments with trust and autonomy.”

- As I mentioned I have worked as Technical Advisor at a IoT based telematics company. (Invense)
- I really enjoyed working on such systems. It was a wonderful experience. Btw it got acquired my one of its clients (Harmonizer)
- So I was looking a opening in similar fields in Germany.
- I came across Emnify and the opening suited my profile.
- We had issues with connecting our IoT devices in middle-east. And Emnify solves that business problem. So I know the business value Emnify brings.
- I wanted to explore how to work and explore on the scale at which Emnify operates.
- And since I have experience in building and scaling systems I applied.
- I had a wonderful experience in Interviews so far. 
  - HR recruiter (Valeria) is very nice and interactive.
- I think I can bring a lot of values to this business and I get to learn a lot about scaling IoT based system.
- Align with values of Emnify like flexibility

I primarily resonate with these 3 values.

Throught my careers, I have solved real world high impact problems. Espcially in Fintech. where every interaction/transactions is important

"Empower People"
Fortunate enough to work with very smart people. 
- I was trusted with very important features 
- I was lucky enough with work with them.
- 


## How to interact with non technical stakeholders?
While interacting with non technical stakeholders I follow.
What → Why → How (optional) Strategy

### What?
I start by clearly explaining what is being built or changed, using plain language.
Ex:
- At Aspire, while building A/B testing framework.
  “We’re building a tool that allows Product Managers to run experiments and compare different user flows to see which performs better.”
- At Scripbox, while migrating to Event Driven Microservice Architecture
  “We’re restructuring the system so updates across services happen more reliably and in near real-time.”

### Why?
Then I explain why it matters, focusing on outcomes and value. - metrics 
Ex: 
- “This helps Product Managers make decisions based on real user behavior instead of intuition, which improves conversion rates.”
- “This reduces system failures during peak usage and helps teams ship features faster, improving both reliability and development speed.” 

### How?
I only go into how if they ask, and I keep it simple and non-technical.
Ex:
- “We provide APIs and a dashboard so teams can easily configure experiments and view results.”
- “We decouple systems so they can operate independently and update each other asynchronously.”

## How to handle ambiguous feature requests? How do you clarify?
CLEAR Framework

ex: At my current role, we had a requirement to enable certain product features—like international payments—only after users met specific compliance requirements, such as submitting valid business documents and going through an Ops verification process (a Maker-Checker flow).

### Context
Understand the bigger picture before anything else.
- Ask: What problem are we solving?
- Ask: Who are the users/stakeholders?

ex:
`
Instead of directly building this as a one-off feature, I first clarified the broader problem:
- We weren’t just enabling international payments
- We needed a consistent way to control feature access based on compliance and verification
`

### Learn
Turn vague ideas into real scenarios.
- Ask: What are the top 2–3 use cases?
- Ask: What does the current flow look like?

ex:
```
I worked with stakeholders (product, ops, compliance) to understand:

- Different feature enablement flows
- Variations in verification requirements
- Current manual processes and pain points

This helped identify that multiple features would need similar lifecycle handling.
```

### Expectations
Define what “good” or "done" looks like.
- Ask: How do we measure success?
- Ask: Deadlines? compliance constraints?
- Get Success metrics + boundaries

```
We defined success as:

- Reducing manual Ops effort
- Enabling reuse across multiple features
- Supporting auditability and compliance requirements
- Allowing clear visibility into feature state transitions
```

### Alternatives
Don’t wait for perfect clarity—drive it.
- Propose 2–3 approaches 
- Highlight trade-offs (complexity, cost, scalability)

```
Instead of building a single feature-specific solution, I proposed:

A tightly coupled implementation for just international payments (quick but not scalable)
A generic framework to handle feature activation workflows across the system

I recommended the second approach, as it would scale better and reduce future development effort.
```

### Reconfirm
Align before building.
“Let me summarize to make sure we're aligned…”
- Problem
- Scope
- Chosen approach
- Trade-offs

```
Before building, I aligned with all stakeholders by summarizing:

- The generalized problem
- The framework approach
- Trade-offs (slightly higher upfront cost, but long-term scalability)
```

### Impact
- Enabled multiple features to reuse the same framework
- Reduced duplication across teams
- Improved compliance tracking and auditability
- Currently being used of Multi Currency onboarding, limited Access Onboarding in SG, HK and US, yield account onboarding in HK
and many more to come.

```
I designed and built a framework called Product Activation Request (PAR), which:

- Provided a standardized lifecycle for feature enablement (request → verification → approval/rejection)
- Exposed APIs for other teams to integrate with
- Supported Maker-Checker workflows for Ops teams
- Allowed easy extension for new features without duplicating logic
```

## How to do you deal with you don't agree with you team members? - conflict resolution
CALM Framework: 
### Context First (Understand before reacting)
- I try to fully understand why they think differently
- Ask clarifying questions:
    “What problem are we optimizing for?”
    “What constraints are we considering?”

### Align on Goals
Bring the conversation back to shared outcomes:
- user impact
- business goals
- system constraints (scale, latency, cost)

### Lay Out Trade-offs (Not opinions)
I present my perspective in terms of:
- pros / cons
- risks
- long-term vs short-term impact

### Move Forward (Disagree & Commit if needed)
- If consensus isn’t reached:
  - I support a decision once made
  - Focus on execution

```
- Situation
  - In one project, the product team wanted to release a feature quickly for our US customers, but engineering felt the implementation wasn’t scalable and needed more time.

- Task
  - As one of the backend engineers, my goal was to help the teams align and still deliver value without creating long-term technical problems.

- Action
  - I scheduled a short discussion with both product and engineering.
  - Instead of debating opinions, I broke the problem into:
    - what is needed immediately for the demo
    - what is needed for long-term production
  - I proposed a phased approach
    - we reused a existing feature to cater to the US customers' requirements
    - plan a proper scalable version in the next sprint
  - This allowed product to meet the deadline without engineering feeling we were creating risky tech debt.

- Result
  - Both teams agreed on the compromise.
  - We delivered the feature on time, and later replaced the temporary implementation with a scalable version.
  - It also improved trust between product and engineering because we handled the disagreement collaboratively rather than emotionally.
```


## Tell me about something you build recently? Walk through entire process?
- Fintech Scripbox
- Invense (IoT) - Case Study is basic - Improve it.

## What challenges did you face while building this?
- Scale 
- Cost
- How dealt

