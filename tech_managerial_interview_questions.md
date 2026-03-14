## ⁠Tell me about yourself
- Hi, I am Sanjeev, I am a software engineer.
- Currently, I work as a senior software engineer at a FinTech called "Aspire FT" based out of Singapore.
- There I am part of the customer onboarding team. I am in-charge creating user and analyst flows for non KYC/KYB compliance.
- I mostly use PHP / Laravel as the main backend. But we also have a new microservice written in Go/Gin. And for DB we use MySQL.
- I provide technical solutions to business requirements working with product managers, designers and frontend engineers.
- I mainly work as an individual contributor, but I also help in onboarding new engineers and helping with code reviews.
- Recently I worked on creating a Proof Of Business Flow for out customers. (Being used by our entities in HongKong, Singapore and US, coming to EU soon).
- Before this role, I worked at an Indian FinTech scaleup called "Scripbox" as a early engineer for 7 years. There I worked on multiple teams.
- First, I started as a frontend engineer, I mainly built customer dashboards and mobile application. I was in this role for 4 years.
- I mostly used React-Redux and React Native to build applications with TypeScript.
- Later, I transitioned to be a full-time backend engineer. I mainly worked on data aggregation and bank statement parser systems.
- I used Elixir\Phoenix and Go\Gin for backend. Used Postres as database with redis.
- I also helped in orienting and onboarding new members to the technical team. Also, helped with interviewing process.
- Its currently one of the largest mutual fund brokers in India. 
- Prior to that I worked at another startup focused on building IoT based solution for parking lot management. I mainly worked as a full stack engineer, where I built dashboards, user app and attendant apps.
- I have done my Bachelor studies in electronics and instrumentation engineering. 
- I also did freelancing projects, building websites and apps for startups and smaller businesses (mainly used PHP/Codeigniter/Wordpress).
- Apart for the work responsibilities, I have worked part time as a technology consultant for a IoT telematics product currently deployed at multiple locations like a mining company called Wier minerals, public infra like Kolkata Railway station.
- In my off-time, I enjoy working on personal projects like economic calculators and other AI based apps. 
- I am also a fitness enthusiast, I do origami crafts and play video games on my free time. 

- Currently, I am exploring opportunities in FinTech across Europe to gain deeper insights for building products for European customers and GDPR compliance.


## Tell me about one of the projects you did recently
- Situation
  - I was working at Scripbox, a digital wealth advisory and investment platform.
  - As a B2C FinTech business, we needed deeper insights into customers’ financial behaviour — such as income, expenses, and savings — to provide better recommendations.
  - This process created significant friction: it required multiple steps and led to a drop-off rate of over 60%, which directly affected onboarding and engagement.

- Task
  - To reduce friction and improve data reliability, we decided to integrate with MFcentral, a centralized financial data repository.
  - The goal was to securely fetch customer financial data through MFcentral’s APIs.
  - This integration required implementing AES-256-CBC encryption, time-based authorization tokens, and handling asynchronous APIs where responses were delivered to a separate endpoint after a delay.

- Action
  I led the backend implementation for this integration.
  - Built a reusable wrapper/SDK to handle encryption, decryption, and authentication at a low level while exposing simple high-level functions for internal teams.
  - Designed a Redis-based exponential backoff mechanism to handle the asynchronous API flow and reliably track request status.
  - Implemented a backup storage mechanism using object storage to persist retrieved financial data for future use and resilience.
  - Ensured the system was modular and reusable so that other teams could leverage the same SDK for additional MFcentral integrations.

- Result
  The integration significantly improved the user experience and business outcomes:
  - Reduced customer drop-off rate from over 60% to around 10%.
  - Improved data retrieval turnaround time to approximately 2–3 minutes.
  - Enabled richer financial insights into customer spending and saving patterns, helping advisory teams provide more personalized recommendations.
  - The SDK I built became a shared internal tool and was later reused by other teams for additional MFcentral-related features.


## How do you resolve conflict? Give an example.

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

## Explain about some of the bugs you fixed or helped to fix recently

Currently, At "Aspire" we triage or bugs based on priority
01. P0: System is unusable or causing major financial/business risk
02. P1: Major functionality broken but system still partially usable
03. P2: Important bug but with a workaround or limited impact
04. P3: Minor issues that don't affect core functionality

One of the recent bugs I fixed this week.

Situation
- Dashboard had “Engagement Items” prompting users to complete actions like POB verification.

Task
- Users clicking the banner were not reaching the correct verification flow.

Action
- Investigated routing → banner pointed to /pob-verification while FE required /pob-verification/missing-files.
- Updated banner configuration to correct route and informed FE team.

Result
- Users could complete the verification process again.
- Immediate mitigation restored functionality until FE implemented the proper fix.

## Explain about the POB Verification feature you did recently? 

### Activation Requests – Core Idea
A workflow engine for customer verification tasks

Each request represents a verification process that moves through defined states until completion.

### Activation Request Lifecycle

#### 1. Entry Trigger
Request created when:
- KYB/KYC completed
- Internal API call
- System event

Initial state:
not_started

#### 2. Customer Action
Customer must provide required information.

Examples:
- Upload documents
- Fill forms
- Provide verification details

State transition:
not_started → processing

#### 3. Internal Review
Verification handled by internal teams.
Intermediate states:
- maker_requested
- checker_requested
- compliance_requested

These mirror existing KYC/KYB review processes.

State transitions controlled via state machine.

#### 4. Terminal State
Request ends as:
- processed
- rejected

Ensures:
- clear workflow completion
- auditability

#### 5. Event Emission

When terminal state reached:
- System emits event containing request result + metadata
- Consumers use event to enable product functionality.

### POB Verification Feature

POB = Proof of Business verification
Required for customers in:
Singapore
Hong Kong

Purpose:
Verify business operational presence before enabling financial services

Workflow

KYB completed
↓
POB Activation Request created
↓
Customer uploads proof-of-business documents
↓
Internal review (maker → checker → compliance)
↓
Request processed or rejected
↓
Event emitted → enable customer functionality

#### My Responsibilities
- Implemented POB workflow using Activation Requests
- Designed state transitions
- Implemented backend APIs
- Integrated document submission + verification flow
- Implemented event emission for downstream systems
- Ensured compatibility with existing KYB/KYC pipelines

### Other Flows Using Same Framework
Framework reused for:
- Hub onboarding via DBS Bank
- Hong Kong Yield account onboarding
- PreNexus verification for US customers
- Upcoming AU customer onboarding

#### Benefit:
- reusable workflow system
- faster onboarding feature development
