# ICEYE — Senior Backend Software Engineer (Tactical Access) — Screen Prep

> Researched 2026-10-02. Sources: Ashby JD (public API), docs.iceye.com, Glassdoor search snippets (Glassdoor blocks direct scraping, so the interview reviews are summarised from snippets, not read in full), press coverage.

---

## 1. The role at a glance

| | |
|---|---|
| Title | Senior Backend Software Engineer (Tactical Access) |
| Org | Business Automation → Engineering, **Tactical Access squad** |
| Reports to | Engineering Lead, Tactical Access squad |
| Location | Espoo, Finland, hybrid **3 days/week in office** |
| Contract | Permanent |
| Salary | **€6,000–8,000 / month gross** (~€72k–96k/yr + Finnish holiday pay) |
| Clearance | **SUPO security screening** (Finnish Security & Intelligence Service). Expect a background check; be ready for nationality/residency questions |
| Relocation | Flights, accommodation, relocation agency |

**Mission in one line:** own the backend that lets *customers* plan and manage **satellite tasking** through Tactical Access, ICEYE's tactical customer-facing product (API + web).

**What they're really hiring for:** a backend owner who treats the **public API as a product**: versioning, backwards compatibility, no breaking changes for integrators, observability, and incident root-causing. It's customer-facing for defence and government users, so reliability and security matter a lot.

### Stack (from the JD)
- **Go** (or a similar typed language) for production APIs
- **PostgreSQL** (+ **PostGIS**, **GeoJSON** listed as a plus)
- **NATS** (sister role mentions **NATS JetStream** with event-driven workers doing *opportunity calculation* and async state sync)
- **AWS, Docker, Kubernetes, Terraform** (on-prem awareness is a plus, since sovereign/defence customers may run air-gapped)
- **Prometheus, Grafana, OpenTelemetry**: they explicitly want you to tell **metrics vs logs vs traces apart during a live incident**
- **OAuth / OIDC / JWT**
- **AI-assisted tools** (Cursor, Devin...). Glassdoor shows they *ask about this*
- Sometimes full-stack (a sister team uses Next.js/React/TS with CesiumJS/deck.gl)

### Expected outcomes (they'll probe each one)
1. Own end-to-end delivery of a significant backend capability, from API design to production, **with documented tradeoffs**
2. Reduce production risk through observability, with **root-cause fixes, not symptom patches**
3. Ship API changes with **no unplanned breaking changes** to integrators
4. Work directly with PMs, **customers**, and other teams
5. Go full-stack when needed without letting backend ownership slip
6. Raise the bar in code review and architecture discussions

### Key competences (scored in every round)
Intellectual Firepower · Passion & Work Ethic · **Ownership & Action** · Team Player · Integrity & Growth Mindset

---

## 2. Interview process

From the JD: **TA Partner Screen → Hiring Manager → Technical Interview → Values & Fit**.
Sister postings and a mirror of this role add a **take-home work sample** before the technical interview, and a **final round with the SVP of Engineering**.

### What Glassdoor says
- ~125 interview reviews. **~46–50% positive**, difficulty **2.7/5**, about **38 days** end to end on average.
- Stage mix: 1:1 (23%), skills test (20%), phone (16%), panel (11%), background check (10%), presentation (10%).
- **Take-home:** one candidate reported a **48-hour take-home**, then a **2-hour follow-up interview just 24h after submission** (theirs was "build a full UI for a provided backend", likely a full-stack role). Budget your time for this if it comes up.
- Questions people reported being asked:
  - "Since you decided to set **http-only cookies** after authentication, what about the other browser tab?" (they dig into *your own* design decisions from the take-home)
  - System design concepts, multithreading/concurrency
  - "**How did you use AI tools** to improve your productivity?" / "How did you use AI in your submission?"
  - "How would you lead inexperienced team members?"
  - "Biggest challenge you've solved?"
- Takeaway: the technical round is mostly **defend-your-design**. Expect "why X, what about Y" follow-ups on everything you build or describe.

### Company reviews (for your own judgement and your questions)
- **3.7/5, 65% recommend** (143 reviews).
- Pros: interesting domain and hard problems, lots of freedom and impact, experienced engineers, decent work-life balance.
- Cons (several recent, around mid-2026): weak or turbulent management, **departments siloed** and slow to help each other, leadership turnover, tight on raises/promotions, unclear career paths.
- → Ask about this tactfully (see §6).

---

## 3. Company context (know this cold)

- **What they do:** run the world's largest **SAR (Synthetic Aperture Radar)** small-satellite constellation. Radar imaging works **day or night, through clouds**, at down to ~25 cm resolution, with high revisit rates.
- **Customers:** mainly **defence / intelligence / ISR** for governments and allies (Tactical Access is this segment). Also dual-use: insurance and natural-catastrophe intelligence, maritime (oil spills, dark vessels), finance.
- **Scale:** 900+ employees. Offices in FI, PL, ES, JP, UAE, GR, US. **62+ satellites launched**, aiming for ~1/week production in 2026 and 100/yr by 2028.
- **Money:** 2025 revenue **~€250M** with **>€100M operating profit**, **€1.5B+ backlog**, targeting ~€1B revenue. **June 2026 Series F: €450M primary (>€1B incl. secondary), led by General Atlantic, valuation >€10B**.
- **Big deals:** **Rheinmetall ICEYE Space Solutions** JV, a **€1.7B German military** SAR constellation contract. **Poland MikroSAR** (€200M, with WZŁ). Many sovereign "own-your-constellation" deals.
- **Why it matters for this role:** sovereign/defence customers mean **on-prem or air-gapped deployments**, strict security, **SUPO** screening, and APIs that partner systems integrate deeply. Breaking those integrations is very costly.

### ICEYE's public Tasking API (ICAS), to sound fluent
Docs: docs.iceye.com/constellation/api/tasking
- **Contract** → limits what you can task (modes, priorities, pricing).
- **createTask**: point of interest (lat/lon, **WGS84**), **acquisition window** (≥24h), **imaging mode**, **priority** (`COMMERCIAL` / `BACKGROUND`), SLA.
- **checkFeasibility**: probability of acquisition. COMMERCIAL tasks are rejected unless there's ~**90%** acquisition probability.
- **Lifecycle:** submitted → ~**20 min feasibility evaluation** → `ACTIVE` or `REJECTED` → `FULFILLED` (products ready). Also expect cancel/failed states.
- **SLA:** standard is **8h** from window start to product availability (covers downlink and processing).
- **listTaskProducts / listTaskDerivatives**: SAR products and derived analytics (e.g. Detect & Classify).
- **Fleet tasking:** the customer asks for an outcome and ICEYE picks the satellite across the whole fleet.
- Versioned API (**v1 → v2**). Ask how they ran that migration.
- Token-based auth (OAuth2 client credentials style).

Domain words to drop naturally: **AOI**, **revisit**, **pass / access window**, **opportunity** (a feasible satellite pass over a target), **downlink / ground station**, **SAR imaging modes** (Spot / Strip / Scan), **TLE/SGP4** (orbit propagation), **ISR**.

---

## 4. Likely technical topics and how to answer

### A. API design & lifecycle (their #1 theme)
- **Versioning:** URI (`/v2/`) vs header/media-type. Additive changes don't need a new version. Breaking changes mean a new version, with a **deprecation policy** (`Deprecation`/`Sunset` headers), a migration guide, and per-client usage metrics so you know who is still on v1.
- **What's breaking?** Removing or renaming fields, changing types or semantics, new *required* inputs, tightened validation, **new enum values** (if clients switch exhaustively, so document that enums are open), changed error codes, changed defaults.
- **Defences:** contract tests / OpenAPI diff in CI (e.g. `oasdiff`), consumer-driven contracts, Postel's law ("tolerant reader"), feature flags per client.
- **Async long-running ops:** the task is a resource with a status. Return `202 Accepted` + `Location`, then let clients **poll** or receive **webhooks** (signed, retried, idempotent).
- **Idempotency keys** on `POST /tasks`: a customer retrying after a timeout must not book two satellite acquisitions. Store key → response in Postgres with a unique constraint.
- **Pagination:** cursor/keyset rather than offset for big, changing lists.
- **Errors:** RFC 9457 problem+json, stable machine-readable codes.
- **Rate limiting / quotas** per contract.

### B. System design: "Design a satellite tasking service" (very likely, prep this one)
Sketch:
1. **API gateway** (authN via OIDC/JWT, per-contract rate limits) → **Tasking API (Go)**.
2. `POST /tasks`: validate (geometry, window ≥24h, contract entitlements) → write task `PENDING` to **Postgres** + an **outbox** row in the same transaction → publish `task.created` to **NATS JetStream**.
3. **Feasibility workers** consume, compute **opportunities** (orbit propagation over the AOI within the window, constraints such as look angle and mode), and score the probability. They publish `task.feasibility_evaluated`.
4. **Planner/scheduler** (another team) books the satellite and emits state changes. The Tasking API consumes them and updates a **state machine** (`PENDING → ACTIVE/REJECTED → ACQUIRED → PROCESSED → FULFILLED`, plus `CANCELED/FAILED`). Guard transitions with optimistic locking (version column) or `SELECT … FOR UPDATE`.
5. Notify the customer through **webhooks** or polling. Products land in **S3**, served via presigned URLs.
- **PostGIS:** store the AOI as `geography(POLYGON,4326)`, GiST index, `ST_Intersects` / `ST_DWithin` queries ("all my tasks over this region").
- **Reliability:** JetStream durable consumers with **at-least-once** delivery, so consumers must be **idempotent** (dedupe on event ID, `Nats-Msg-Id` for publish dedup). DLQ via max-deliver plus an advisory. Ack after commit.
- **Ordering:** per-task ordering via subject `tasks.<id>.events`, or a version number so stale events are dropped.
- **Multi-tenancy & security:** row-level contract isolation, audit log for every action (defence customers), least privilege.
- **On-prem variant:** same services in Helm charts, NATS clustered locally, no managed AWS dependencies. Terraform modules vs cloud-only services.
- **Observability:** RED metrics per endpoint, a trace from the HTTP request across the NATS hop (propagate W3C trace context in message headers), SLOs such as "p99 createTask < 500ms" and "feasibility result < 20 min".

### C. NATS / JetStream (know the basics)
- Core NATS: fire-and-forget pub/sub, subjects with wildcards (`*`, `>`), **queue groups** for load-balanced consumers, request/reply.
- **JetStream:** persistent **streams**, **consumers** (durable, push/pull; pull is preferred for workers), ack policies (explicit), `AckWait`, `MaxDeliver`, redelivery, **exactly-once-ish** via publish dedup window + idempotent consumer + double-ack. KV and Object Store are built on top.
- vs Kafka: simpler ops, subject-based routing, lighter. Kafka has partition-ordered logs and a stronger replay/ecosystem.

### D. Observability & incidents (they explicitly test this)
- **Metrics** = *is something wrong, and how much?* (aggregated, cheap, alerting). **Traces** = *where in the request path?* (latency breakdown across services). **Logs** = *why exactly?* (detailed events, error context). Link them with trace IDs in logs and exemplars on metrics.
- Alert on **symptoms / SLO burn rate**, not causes. Avoid alert fatigue.
- Prometheus: counter/gauge/histogram. `rate()`, `histogram_quantile()`. Avoid **high-cardinality** labels (no task IDs as labels!).
- Live incident walk-through: alert → dashboard (RED/USE) → narrow to service → trace exemplar → logs → mitigate (rollback/flag) → **root cause** → postmortem + fix + regression test + new alert.
- Have a **real incident story** ready (see §5).

### E. Go depth (expect questions or code reading)
- Goroutines, channels, `select`, **`context`** propagation and cancellation/timeouts, `sync.WaitGroup`, `errgroup`, `sync.Mutex` vs channels, worker pools, the race detector, goroutine leaks.
- Error handling: wrapping (`%w`, `errors.Is/As`), sentinel vs typed errors.
- Interfaces (small, defined at the consumer), testing with table-driven tests, `httptest`, testcontainers for Postgres/NATS.
- Graceful shutdown (`signal.NotifyContext`, `server.Shutdown`, drain NATS subscriptions).
- `database/sql` / `pgx` connection pools, transactions, avoiding N+1 queries.
- Multithreading came up on Glassdoor: be ready for **data races, deadlocks, mutex vs atomic**.

### F. Postgres
- Indexing (B-tree, GiST for geo, partial and composite indexes), `EXPLAIN ANALYZE`, isolation levels, row locking, **outbox pattern**, zero-downtime migrations (expand → migrate → contract; add nullable column, backfill, then enforce; `CREATE INDEX CONCURRENTLY`).

### G. Auth (OAuth/OIDC/JWT)
- Flows: **client credentials** (machine-to-machine API integrators, the main one for Tactical Access), **auth code + PKCE** (web UI).
- JWT validation: signature via **JWKS** (cache it, handle key rotation), `iss`, `aud`, `exp`, `nbf`. Scopes/claims → contract permissions. Short-lived access tokens + refresh tokens. JWTs are hard to revoke, so keep lifetimes short or use introspection.
- The **http-only cookie / other tab** question: http-only cookies are sent automatically by every tab on the same origin, so sessions are shared. Discuss **CSRF** (SameSite, CSRF tokens), logout sync across tabs (BroadcastChannel / storage events), and silent refresh.

### H. Kubernetes / AWS / Terraform (breadth, not depth)
- Deployments, readiness vs liveness probes, HPA, ConfigMaps/Secrets, rolling updates, resource requests/limits. Terraform state/locking and modules. EKS, RDS, S3, IAM roles for service accounts.

### I. AI-assisted engineering (they *will* ask)
Have a concrete, honest answer: how you use Claude Code / Cursor (scaffolding, test generation, exploring unfamiliar code, reviewing diffs), **how you verify output** (tests, reading every line, not trusting it on security/concurrency), and where you *don't* use it. Include one specific example with a time saved.

---

## 5. Map your experience → their outcomes

| They want | Your story |
|---|---|
| Go production backend | **Aspire**: new Go/Gin microservice for onboarding/KYC flows, alongside the PHP/Laravel monolith. Talk about what you built, how it's deployed and monitored, and the strangler-pattern split from the monolith |
| Async APIs, reliability | **Scripbox × MFcentral**: async API with delayed callbacks, Redis-based exponential backoff/status tracking, object-storage backup, reusable SDK. **Drop-off 60% → 10%**. This is the same shape as satellite tasking (submit → async status → result), so say so explicitly |
| API as a product / SDK for others | The MFcentral SDK was reused by other teams. Talk about how you kept its interface stable |
| Observability / incident root cause | Prep **one incident story** (STAR): symptom → how you diagnosed it (metrics/logs/traces) → root cause → permanent fix → what you added (alert, test) |
| Backwards compatibility | Any time you changed an API/schema used by mobile apps (Scripbox had mobile clients you couldn't force-update, a perfect example) or by other teams |
| Full-stack when needed | 4 years frontend at Scripbox (dashboards, mobile), full-stack at the IoT parking startup |
| Raising the bar / mentoring | Onboarding engineers, code reviews, interviewing at Scripbox/Aspire |
| Defence / hardware / geo flavour | IoT telematics consulting (Weir Minerals, Kolkata railway station), parking-lot IoT. Electronics & Instrumentation degree, so you're comfortable with sensors/RF-ish domains |
| Compliance / security mindset | KYC/KYB compliance flows at Aspire map well to "security & quality best practices in a customer-facing system" |

**Gaps to prepare for honestly:** NATS (compare to the Redis/queues you've used, and read the JetStream docs), PostGIS (do a quick hands-on exercise), Prometheus/OTel if you haven't run them yourself. Framing: "I haven't used X in production, but here's the equivalent I have used and how I'd approach X."

### Rewrite "Tell me about yourself" for ICEYE
Drop the "exploring FinTech in Europe" ending. Instead say: *backend engineer focused on reliable, customer-facing APIs in Go, with async/integration-heavy systems; a background in electronics & IoT means I'm drawn to physical-world systems; ICEYE's tasking API is exactly the async, reliability-critical, customer-facing kind of problem I enjoy.*

**Why ICEYE?** Mission (real-world security and disaster response), a hard and unusual domain (space + radar), API-as-product ownership, a strongly growing and *profitable* company, and the move to Finland.

---

## 6. Questions to ask them
1. How is Tactical Access split between the public API and the web app? Who are the main integrators (partner C2 systems, government ground segments)?
2. How did the v1 → v2 Tasking API migration go, and what's your deprecation policy today?
3. How much of Tactical Access has to run **on-prem / air-gapped** for sovereign customers, and how does that constrain architecture?
4. How does the squad interact with the planning/scheduling teams? Where's the boundary (NATS contracts)? *(Gently probes the "silos" theme from Glassdoor.)*
5. What does on-call look like, and what was the last significant incident?
6. What would "great" look like for this hire at 6 and 12 months?
7. How does the SUPO screening affect start date and timeline?
8. How are engineers grown and promoted here? *(Probes the career-path concern.)*

---

## 7. Final-day checklist
- [ ] Skim docs.iceye.com Tasking API (createTask, feasibility, statuses)
- [ ] Whiteboard the tasking-service design (§4B) once, out loud, in 30 min
- [ ] Write a tiny Go worker: JetStream pull consumer + Postgres idempotent upsert + graceful shutdown
- [ ] PostGIS: create a `geography` column, GiST index, run one `ST_Intersects` query
- [ ] Prepare STAR stories: incident/root cause, backwards-compatible API change, disagreement with PM, mentoring
- [ ] Prepare an AI-tools answer with a concrete example
- [ ] If there's a take-home: write a short **DECISIONS.md / tradeoffs** section (they want *documented tradeoffs*) and be ready to defend every choice

---

### Sources
- JD: https://jobs.ashbyhq.com/iceye/637982df-5c1e-437a-81ee-3e234f75a732
- Sister role (Tasking & Planning): https://www.dreamworkhq.com/job/a8b4057d-e049-45e6-8e1d-80f6e7c00478
- Glassdoor interviews: https://www.glassdoor.com/Interview/ICEYE-Senior-Software-Engineer-Interview-Questions-EI_IE2278536.0,5_KO6,30.htm · https://www.glassdoor.com/Interview/ICEYE-Interview-Questions-E2278536.htm
- Glassdoor reviews: https://www.glassdoor.com/Reviews/ICEYE-Reviews-E2278536.htm
- Tasking API docs: https://docs.iceye.com/constellation/api/tasking/
- Funding: https://thedefensepost.com/2026/06/12/iceye-satellite-funding-round/amp/ · https://www.insurancejournal.com/news/international/2026/03/13/861819.htm · https://sacra.com/c/iceye/


# Interview Questions
## ⁠Tell me about yourself
- Hi, I am Sanjeev, I am a software engineer.
- I currently work as a senior software engineer at Aspire FT. I work on the onboarding team.
- I am in-charge creating user and analyst flows for non KYC/KYB compliance.
- I provide end to end technical solutions to business requirements working with product managers, designers and Frontend engineers. 
- I mainly work as an individual contributor, but I also help in onboarding new engineers and helping with code reviews.

- Recently I have been working on expanding business for European market (Netherlands, France and Germany) @ Aspire

- This is my current role. I have been in this role for last 1.5 years. 
- Before this, I worked at a Indian Fintech called Scripbox for 7 years, in wealth management space.
- I worked various roles, as a frontend engineer and later as a backend engineer.
- Prior to that worked in IOT space in Parking LOT management.

- I am currently exploring backend roles in Europe, in **regulated, mission-critical domains** — fintech, defence, space — where reliability, security and compliance aren't optional.
- That's what I've been doing in fintech: compliance-heavy flows, integrations with external partners, and APIs where a mistake has real consequences.
- And I've always been drawn to systems connected to the physical world — my degree is in electronics & instrumentation and I started out in IoT.
- ICEYE sits right at that intersection: a customer-facing tasking API, where reliability matters, for satellites. That's why this role stood out to me.


## How do you resolve conflict? Give an example.

- Situation (keep to ~20 sec)
  - At Aspire we were expanding into the US through a beta program with early customers.
  - Product had a demo with 5 beta customers in 2 weeks showing US accounts (debit, credit, yield) — but our new US banking partner wasn't integrated yet. A proper integration was estimated at 4 weeks.
  - Product wanted the full flow built for the demo; engineering pushed back that rushing a new partner integration would leave us with something fragile in a regulated money flow. It had become a deadlock.

- Task
  - I was one of the backend engineers on the integration. My goal: get both sides to a plan that hit the demo date without shipping throwaway code we'd be stuck with.

- Action
  - **Reframed the debate.** I got product and engineering in one short meeting and split the requirement into two lists: what the demo *actually* has to show vs. what production needs (compliance, reconciliation, failure handling, scale). It turned out the demo needed **[the account-opening + balance/yield views]** — not money movement at scale.
  - **Found a faster path.** Our existing Hong Kong banking partner already offered a comparable US product, and we had a working integration with them. I proposed powering the demo on that.
  - **Made the shortcut cheap to replace — the key decision.** I put a **banking-partner interface (adapter)** in front of the provider, so the rest of the system called our own contract (`OpenAccount`, `GetBalance`, …), not the partner's API. The demo ran on an HK-partner adapter; the real US partner would be a second adapter behind the same interface — no changes to callers.
  - **Kept it honest.** I flagged that customers must know this was a pilot environment; product positioned it as a preview, and we kept demo data isolated from production. *(Confirm this is what actually happened.)*
  - **Wrote the tradeoff down.** A short doc listing what was deliberately skipped (e.g. [reconciliation, retries/idempotency, monitoring]) and the follow-up tickets, which I got scheduled into **[sprint X]** before the demo — so it wasn't "we'll fix it later" debt.

- Result
  - Demo delivered on time **[2 weeks]**; **[outcome — e.g. N of M beta customers moved forward / feedback that changed the roadmap]**.
  - The real US partner integration went in as a new adapter in **[4 weeks]**, with **[no / minimal]** changes to the rest of the codebase.
  - The adapter pattern became how we added partners afterwards **[e.g. reused for NL/DE — only if true]**.
  - Product and engineering got a repeatable way to handle deadline-vs-quality disputes: separate "demo" from "production" requirements explicitly instead of arguing in the abstract.

- Lesson (one line to close)
  - Most product-vs-engineering conflicts aren't really disagreements about the goal — they're two different requirement lists mixed together. Splitting them, and making shortcuts *designed to be replaced*, resolves most of them.

- Likely follow-ups — prep answers
  - *"What would you have done if product refused the phased approach?"* → escalate with the written tradeoff doc to the decision owner (EM/Head of Product); disagree-and-commit once decided, but get the risk acknowledged in writing.
  - *"What did you skip, and did any of it bite you?"* → be specific and honest.
  - *"Was it ethical to demo on a different partner?"* → yes because customers knew it was a preview, and the capability shown was what we'd deliver.
  - *"How did you make sure the follow-up actually happened?"* → tickets scheduled before the demo, you owned them.
  - *"Was there anyone you personally disagreed with?"* → have one sentence on the engineer/PM you had to convince and how.



## Questions to Ask the Interviewer

> **This round = TA Partner (recruiter) screen.** They can't answer deep tech/architecture questions — save those for the Hiring Manager and Technical rounds (§6). Ask about **role context, process, logistics, and eligibility**. Pick **4–5**, not all. Write down the answers — they shape your prep for the next rounds.

### 1. The role & team (shows genuine interest)
- What's the story behind this opening — is it a **new position** as Tactical Access grows, or a backfill?
- How big is the Tactical Access squad today, and what's the mix (backend / frontend / product)?
- What made the hiring manager open this as a **backend-focused** role, given the product is both API and web?
- What does the hiring manager say is the **biggest thing they need this person to own** in the first 6 months?

### 3. Eligibility, security & relocation (important for you — ask clearly)
- The posting mentions **SUPO security screening**. What does it involve, how long does it usually take, and are there **nationality or residency requirements** I should know about up front? *(Ask this early — better to know now than after four rounds.)*
- Does ICEYE sponsor the **Finnish residence/work permit** (e.g. specialist permit / EU Blue Card), and roughly how long does that take from offer to start?
- Can you tell me more about the **relocation package** — is there support for family, temporary housing, and the permit process?
- How flexible is the **3-days-in-office** arrangement, especially during the first months?

### 4. Culture & growth (light touch — recruiter-appropriate)
- How would you describe the culture in the engineering org, and what kind of people thrive in it?
- How are engineers supported in growing — is there a defined career ladder (e.g. Senior → Staff)?
- With the recent funding and growth, how is the engineering organisation changing over the next year?

### 5. Close the call
- Is there anything in my background you'd like me to clarify, or any concern I can address now?
- What are the next steps, and when can I expect to hear back?

### Be ready to answer (recruiters always ask)
- **Salary expectation:** the band is €6,000–8,000/month gross. Give a number inside it, aimed high given seniority: *"Based on the posted range and my experience, I'm targeting around €6000- 7000/month, open depending on the overall package."*
- **Notice period / earliest start date:** [X weeks at Aspire].
- **Work authorisation:** current nationality/residency, need for a Finnish permit.
- **Why ICEYE / why this role** — the 30-second version from §5.
- **Why leaving Aspire** — keep it positive: you want mission-critical, customer-facing API ownership in Europe.
- **Other processes in flight** — be honest but brief ("sa couple of processes in progress, ICEYE is a top choice").
- **Comfortable with Espoo, 3 days onsite, and security screening?** → clear yes.