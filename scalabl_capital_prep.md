# Scalable Capital — Interview Prep

Role: (Senior) Full-Stack Engineer (TypeScript, Kotlin/Java, m/f/x)
Interview stage: Initial interview — goal is to get to know you, evaluate skillset, learn about past experience.

## Company Brief

Scalable Capital is a Munich-based fintech offering a digital investment and banking platform, operating as a regulated bank (Scalable Capital Bank GmbH) across Europe. Core products:

- **Broker** — self-directed trading of stocks, ETFs, crypto (low-cost, mobile-first).
- **Wealth Management** — automated, professional portfolio management (robo-advisor style).
- **Savings** — overnight and fixed-term savings accounts, kids savings accounts.
- **Credit** — lending up to €250,000 against portfolio/assets.
- **Retirement** — pension products with government subsidies.

Business model: trading fees, wealth management fees, interest margin on savings/credit. Positioned as a low-cost, accessible alternative to traditional banks/brokers, targeting retail investors and savers. Operates in a regulated banking environment (compliance, security, data privacy are first-class concerns).

## Job Description Summary

**Responsibilities**
- Build/maintain client-facing apps, internal tools, APIs, and microservices across the stack.
- Work across frontend, backend, async queues, and cloud infra.
- Collaborate with Product, Operations, Design, Data.
- Write clean, testable, maintainable code — strong emphasis on security, data privacy, operational reliability (regulated environment).
- Contribute to architecture, testing, observability; senior scope includes mentoring and technical direction.

**Required Skills**
- TypeScript, React/Next.js, Node.js/NestJS.
- Kotlin or Java (production backend experience).
- REST APIs, GraphQL, auth patterns (JWT).
- AWS, Docker, Git, CI/CD, observability.
- CS degree or equivalent practical experience.
- Pragmatic trade-off thinking, professional English.

**Nice to have**
- Distributed systems / event-driven architecture.
- Python or other backend languages.
- Experience in regulated/high-security environments.
- Interest/experience with AI-powered features.

## Prep Plan for This Round

Map your background to each required area with a concrete example (project, scale, impact):

1. **TypeScript / React / Next.js** — a frontend project you led or built end-to-end.
2. **Node.js / NestJS (or backend generally)** — API/microservice you designed; how you handled testing, scaling.
3. **Kotlin / Java** — most relevant production backend work; be ready to speak to even if it's not your primary stack.
4. **REST/GraphQL/Auth** — a system where you designed the API contract and auth flow.
5. **Cloud/AWS/Docker/CI-CD** — deployment pipeline or infra you owned.
6. **Regulated/security-sensitive work** — any fintech, healthtech, or compliance-heavy experience; data privacy handling.
7. **Cross-functional collaboration** — example working closely with Product/Design/Data.
8. **Trade-offs & pragmatism** — a story where you chose a "good enough" solution over a perfect one, and why.
9. **Mentoring/technical direction** (if senior-level expectations apply) — examples of guiding other engineers or driving architecture decisions.

Also prepare:
- A concise "walk me through your background" narrative (2-3 min).
- 2-3 questions to ask them (team structure, tech stack decisions, how they balance regulatory constraints with shipping speed, AI adoption plans).
- Why Scalable Capital / why fintech — genuine motivation tied to their mission (democratizing investing/low-cost access).



# Interview Questions
## ⁠Tell me about yourself
- Hi, I am Sanjeev, I am a software engineer.
- Currently, I work as a senior software engineer at a FinTech called "Aspire FT" which is based out of Singapore.
- There I am part of the Growth Platform team. I am in-charge creating user and analyst flows for non KYC/KYB compliance.
- I provide technical solutions to business requirements working with product managers, designers and Frontend engineers.
- I mainly work as an individual contributor, but I also help in onboarding new engineers and helping with code reviews.
- Before this role, I worked at an Indian FinTech scaleup called "Scripbox" as a early engineer for 7 years. There I worked on multiple teams.
- First, I started as a frontend engineer, I mainly built customer dashboards and mobile application. I was in this role for 4 years.
- Later, I transitioned to be a full-time backend engineer. I mainly worked on data aggregation and bank statement parser systems.
- I also helped in orienting and onboarding new members to the technical team. Also, helped with interviewing process.
- Prior to that I worked at another startup focused on building IoT based solution for parking lot management. I mainly worked as a full stack engineer, where I built dashboards, user app and attendant apps.

- I recently been working on expanding business for European market (Netherlands, France and Germany) @ Aspire. 

- Currently, I am exploring opportunities in FinTech across Europe to gain deeper insights for building products for European customers and GDPR compliance.


## How do you resolve conflict? Give an example.

- Situation
  - In one project, the product team wanted to release a feature quickly for a client demo, but engineering felt the implementation wasn’t scalable and needed more time.

- Task
  - As one of the backend engineers, my goal was to help the teams align and still deliver value without creating long-term technical problems.

- Action
  - I scheduled a short discussion with both product and engineering.
  - Instead of debating opinions, I broke the problem into:
    - what is needed immediately for the demo
    - what is needed for long-term production
  - I proposed a phased approach
    - build a simpler version for demo using a temporary solution
    - plan a proper scalable version in the next sprint
  - This allowed product to meet the deadline without engineering feeling we were creating risky tech debt.

- Result
  - Both teams agreed on the compromise.
  - We delivered the demo on time, and later replaced the temporary implementation with a scalable version.
  - It also improved trust between product and engineering because we handled the disagreement collaboratively rather than emotionally.

## Questions to Ask the Interviewer (Software Engineer)

**Team & day-to-day**
- What does the team structure look like — how are engineers split across Broker, Wealth Management, Savings, Credit, Retirement? Is it feature teams or platform/shared-services teams?
- What does a typical sprint/week look like for you — how much is new feature work vs. maintenance, incident response, or paying down tech debt?
- How is on-call handled, and how often does it come up given the regulated/banking context?

**Tech stack & architecture**
- You use both TypeScript and Kotlin/Java — how is that split in practice? Is it per-service, or do engineers move between frontend and backend/JVM work regularly?
- Is the backend a monolith, modular monolith, or microservices? How many services roughly, and how do teams manage ownership boundaries?
- How do you handle communication between services — REST, GraphQL, async/event-driven (queues, Kafka, etc.)? Where does each fit?
- What does your CI/CD pipeline look like — how long from merge to production, and how much is automated (tests, canary/rollout, rollback)?
- What's observability like day-to-day — what do you reach for first when something breaks in production?

**Regulatory / security tension**
- How do you balance shipping speed with the compliance/audit requirements of being a regulated bank? Is there a formal review gate, or is it built into normal engineering process?
- How involved are engineers in security reviews or threat modeling for new features, versus that being owned by a separate security/compliance team?
- Has a compliance or regulatory requirement ever significantly reshaped a technical design you were working on? What was that like?

**AI adoption**
- The listing mentions interest in AI-powered features — is that exploratory right now, or are there AI features already in production? What's the team's approach to using AI tools (Copilot/Claude/etc.) in the actual engineering workflow?

**Growth & culture**
- What does career growth look like for a senior engineer here — more scope on a single team, or movement across the Broker/Wealth/Savings/Credit domains?
- What's one thing about working here that surprised you after joining?
- What's the biggest engineering challenge the team is tackling right now?
