## ⁠Tell me about yourself
- Hi, I am Sanjeev, I am a software engineer.
- Currently, I work as a senior software engineer at a FinTech called "Aspire FT" based out of Singapore.
- There I am part of the customer onboarding team. I am in-charge creating user and analyst flows for non KYC/KYB compliance.
- I provide technical solutions to business requirements working with product managers, designers and Frontend engineers.
- I mainly work as an individual contributor, but I also help in onboarding new engineers and helping with code reviews.
- Before this role, I worked at an Indian FinTech scaleup called "Scripbox" as a founding engineer for 7 years. There I worked on multiple teams.
- First, I started as a frontend engineer, I mainly built customer dashboards and mobile application. I was in this role for 4 years.
- Later, I transitioned to be a full-time backend engineer. I mainly worked on data aggregation and bank statement parser systems.
- I also helped in orienting and onboarding new members to the technical team. Also, helped with interviewing process.
- Its currently one of the largest mutual fund brokers in India. 
- Prior to that I worked at another startup focused on building IoT based solution for parking lot management. I mainly worked as a full stack engineer, where I built dashboards, user app and attendant apps.
- I have done my Bachelor studies in electronics and instrumentation engineering. 
- I also did freelancing projects, building websites and apps for startups and smaller businesses.
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