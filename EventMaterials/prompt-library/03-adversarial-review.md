# 03 — Adversarial Review Prompts

*Attack your design, expose blind spots, and catch assumptions before production.*

---

## Section Purpose

Architects are optimists. They design for the happy path. AI can play the role of a skeptical reviewer — exposing hidden assumptions, failure modes, and risks.

These prompts help you:
- **Identify hidden assumptions** in your architecture (team skills, vendor capabilities, operational practices)
- **Find scaling risks** (what breaks when load 10x? 100x? Data grows?)
- **Expose security gaps** (who can access what? What's the blast radius if a component fails?)
- **Challenge operational assumptions** (is observability actually achievable with this design?)
- **Find integration risks** (what could go wrong when components talk to each other?)
- **Simulate a skeptical Architecture Review Board** (CTO, CISO, SRE, Finance challenging your choices)

**Thesis Connection:** AI enables critique. Here it attacks your architecture so architects can catch blind spots before they reach production.

---

## Prompt Catalog

## Prompt Cards (Copy-Paste Ready)

---

### Card 1: Failure-Mode Architecture Review

**When to Use:** After you've drafted architecture; before design review  
**When NOT to Use:** Don't use on exploratory sketches (components must be clear); run Sections 01-02 first.  
**Phase:** Architecture Exploration → Adversarial Review  
**Risk Focus:** Hidden assumptions, operational gaps, failure modes

#### Prerequisites
- Architecture description (text or diagram: component names, responsibilities, data flows)
- Constraints: team size, SLA, latency targets, scale, environment
- Current design decision(s)

**Validation Rule:** If architecture description is missing details on any component boundary or data flow, output `[INCOMPLETE: <missing detail>]` and list required clarifications. Do not proceed with assumptions.

#### Inputs Required
```
Component List:
- Service/component name
- Core responsibility
- Data inputs/outputs
- Failure mode handling (or note if undefined)

Constraints:
- Team size: [N people]
- Max concurrent users: [number]
- SLA uptime target: [e.g., 99.9%]
- Latency target: [p99 acceptable latency or N/A]
- Environment: [single-region, multi-region, on-prem, hybrid]

Known Risks Already Mitigated: [List any you've already thought through]
```

#### The Prompt

```text
You are an adversarial architecture reviewer. Your job is NOT to be polite.
Attack this architecture and find assumptions the team didn't realize they were making.

[PASTE CONSTRAINTS SECTION HERE]

Architecture Components:
[PASTE COMPONENT LIST HERE]

For each component, answer:

1. WHAT ASSUMPTIONS IS THIS COMPONENT MAKING?
   - About upstream components (what data it receives, format, frequency, accuracy)?
   - About downstream components (how they'll consume output)?
   - About the team (skills, on-call rotation, incident response)?
   - About the environment (network latency, disk performance, vendor availability)?
   
   Mark each assumption as [ASSUMED] and state confidence (1-5) if it's not explicitly validated.

2. WHAT BREAKS IF THIS COMPONENT FAILS?
   - Trace the failure chain: If [component] fails, what other components are affected immediately (next 1 second)? In 1 minute? In 1 hour?
   - Which failures are silent (no error signal) vs. loud (error propagates)?
   - Which failure chains violate the stated SLA?
   
   Format: "[Component Y] → [Component Z] fails because [specific reason]. Impact: [data loss / latency spike / financial impact]"

3. WHAT OPERATIONAL PRACTICE IS THIS COMPONENT ASSUMING?
   - On-call rotation? Specific runbook? Monitoring?
   - Does the team actually DO this today?
   - If not, flag as [OPERABILITY RISK: <specific practice required>]

4. SCALING BLIND SPOTS (given max concurrent users = [constraint]):
   - Which component will hit its limit first if load doubles? 10x?
   - What's the cost/performance tradeoff if it does?
   - Any data structure that becomes problematic as data grows (e.g., in-memory cache, single table)?

5. DEPENDENCY RISKS:
   - Does this component depend on external services (cloud provider, 3rd-party API, vendor)?
   - What's the blast radius if that dependency goes down?
   - Are there fallbacks, or does the entire system fail?
   - Mark as [VENDOR LOCK-IN RISK] if applicable.

OUTPUT FORMAT:
For each component:
- COMPONENT: [name]
- ASSUMPTIONS: [ASSUMED: assumption 1 (confidence X/5); ASSUMED: assumption 2 (confidence Y/5); ...]
- FAILURE CHAINS: [specific failure sequences with impact]
- OPERABILITY RISKS: [list or "NONE"]
- SCALING RISKS: [description of bottleneck and impact; or "ACCEPTABLE UNTIL load reaches X"]
- DEPENDENCY RISKS: [description; or "NONE"]

SUMMARY SECTION (end of output):
- Top 3 most critical assumptions (if any fail, architecture fails)
- Top 3 most concerning failure chains
- Top 2 operability gaps (what the team needs to do to run this safely)

VALIDATION RULES:
- Every risk must reference a specific component, not generic concerns
- Risks must be traceable to stated constraints or known failure patterns (no invented scenarios)
- If you cannot verify an assumption against the input, mark it [UNVERIFIED] and note why it matters
```

#### Customization Knobs

**By Architecture Style:**
- **Monolith:** Focus on deployment coupling + scaling bottleneck (single shared database)
- **Microservices:** Focus on integration risks + distributed failure chains + data consistency
- **Event-Driven:** Focus on ordering guarantees + message loss + replay scenarios
- **Serverless:** Focus on cold-start impact + cost surprises + vendor lock-in

**By Operational Maturity:**
- **Startup (no SRE):** Emphasize operability risks (what can actually be run?)
- **Mid-scale:** Balance scaling + operational + dependency risks equally
- **Enterprise:** Add governance/compliance risks to the review

**By Domain:**
- **E-commerce:** Emphasize data consistency (inventory, payment) + scaling (Black Friday traffic)
- **Healthcare:** Emphasize HIPAA data flow risks + audit trails
- **Financial:** Emphasize transaction consistency + auditability

#### Example Input & Output

**Good Output (Grounded):**
```
COMPONENT: Order Processing Service
ASSUMPTIONS:
- ASSUMED: Payment gateway has <100ms latency (confidence 2/5 — not verified; SaaS vendor claim)
- ASSUMED: Primary database replication lag is <1 second (confidence 3/5 — needs verification test)
- ASSUMED: Team has on-call rotation (confidence 1/5 — not mentioned in requirements)

FAILURE CHAINS:
- If payment gateway is unavailable: Order Processing queues requests → queue grows unbounded → memory exhausted → service crashes in ~10 minutes. No fallback. Impact: All orders fail. Violates 99.9% SLA immediately.
- If primary DB replica fails: Read traffic redirects to primary; primary becomes bottleneck; latency spikes to >1s; slow reads cascade to Order Service; timeouts cascade to API → SLA breach in 2 minutes.

OPERABILITY RISKS:
- [OPERABILITY RISK: No circuit breaker on payment gateway calls. If vendor is down, requests block indefinitely. Requires rate-limiting + timeout strategy.]
- [OPERABILITY RISK: No alerting on queue depth. Team won't know queue is backing up until crashes occur.]
```

**Weak Output (Hallucinated/Generic):**
```
ASSUMPTIONS: This architecture assumes scalability and resilience (generic, not specific)
FAILURE CHAINS: Various components could fail in production (no specific chain)
OPERABILITY: Needs good monitoring (generic advice, not grounded in design)
```

#### Troubleshooting

- **Output is too generic** ("scalability is hard"): Re-run with additional specificity: "For each component, explicitly name which constraint it depends on. If a risk doesn't reference a named constraint, don't output it."
- **Output misses real risks you know about:** Prompt likely didn't include all constraints. Add missing SLA, load, or team-skill constraint; re-run.
- **Output invents unrealistic failure scenarios:** Prompt the AI to "Only consider failure modes that have happened to real systems or are documented in known case studies. Don't invent hypothetical scenarios."

---

### Card 2: Catastrophic Backcast Review

**When to Use:** After architecture is drafted; forces "assume it broke in production" thinking  
**When NOT to Use:** Skip if you're still in early exploration; use this after you've committed to an approach.  
**Phase:** Architecture Exploration → Adversarial Review  
**Risk Focus:** Failure propagation, cascading effects, missed early warnings

#### Prerequisites
- Current architecture (same format as Card 1)
- Operational constraints: team size, SLA, incident response time
- Known incidents in similar systems (optional but valuable)

#### The Prompt

```text
Assume your production system has failed catastrophically. It's 3 AM. The system is down.

Architecture:
[PASTE ARCHITECTURE DESCRIPTION HERE]

Operational Constraints:
- Team size: [N people]
- SLA uptime: [percentage]
- Incident response time to first troubleshooting action: [minutes]
- On-call rotation: [yes/no; if yes, how many rotations?]

For this failure scenario, work backwards:

1. MOST LIKELY FAILURE START:
   - Given your components and dependencies, what is the MOST likely first thing to fail?
   - Rank top 3 failure origins by probability (based on component complexity, external dependency count, known failure patterns in similar systems).
   - For each, explain WHY it's likely to fail (e.g., "Database connection pooling has no upper bound; under sustained traffic spike, pool exhausts").

2. FAILURE CASCADE (How it propagates):
   - Take the most likely failure origin
   - Trace the failure chain: What fails next? What fails after that?
   - At each step, ask: Is there a circuit breaker? A fallback? A graceful degradation step? Or does failure cascade silently?
   - Timeline: At what point does the team get alerted? (What takes longer than team response time?)

3. WHAT EARLY WARNING SIGNS WERE MISSED?
   - In the hours/minutes/seconds BEFORE catastrophic failure, what metrics would have shown the problem?
   - Did the architecture include monitoring for those metrics?
   - If yes, what's missing? (Alert rules? Thresholds? Someone on-call to respond?)
   - If no, add [MISSING OBSERVABILITY: <metric>]

4. WHICH DESIGN DECISIONS MADE THIS FAILURE WORSE?
   - Point to specific architecture choices that: 
     a) Made failure MORE likely (e.g., single point of failure)
     b) Made failure cascade faster (e.g., no circuit breaker)
     c) Made failure harder to detect (e.g., silent failure mode)
     d) Made recovery slower (e.g., long TTL on caches preventing cache invalidation)

5. WHAT DID THE TEAM NOT KNOW THEY WERE ASSUMING?
   - About team skill (e.g., "Assumes team knows how to debug distributed system race conditions")
   - About environment (e.g., "Assumes database failover is automatic; it's not in this provider")
   - About vendor behavior (e.g., "Assumes vendor rate-limits are soft; they're hard limits")

OUTPUT FORMAT:

CATASTROPHIC FAILURE SCENARIO:
[Specific failure origin] → [Specific cascade] → [System down]

MOST LIKELY ORIGIN:
- Failure: [specific]
- Why: [reason grounded in component design or constraint]
- Probability: [justification]

FAILURE CASCADE:
[Step-by-step chain with timestamps and component impacts]

EARLY WARNING SIGNS:
- [Metric 1 that should spike]: [when it should spike relative to catastrophic failure]
  - Monitored? [yes/no]
  - Alert configured? [yes/no]
  - Someone on-call to respond? [yes/no]
- [Metric 2]: ...

DESIGN DECISIONS THAT ACCELERATED FAILURE:
1. [Decision 1 and its consequence]
2. [Decision 2 and its consequence]

UNVALIDATED ASSUMPTIONS:
- ASSUMED: [assumption] (validation status: NOT VERIFIED)
- ASSUMED: [assumption] (validation status: NOT VERIFIED)

RECOVERY PLAN (What's missing):
- To prevent: [specific mitigation]
- To detect faster: [specific monitoring]
- To recover faster: [specific runbook or automation]
```

#### Customization
- **Startup (short MTTR patience low):** Focus on "what's the fastest way to restore service?"
- **Enterprise (long SLA windows):** Focus on "what should have prevented this?"

#### Example
**Good Backcast:**
```
CATASTROPHIC FAILURE: Database connection pool exhausts → All queries queue → Timeouts cascade → API returns 500 → Frontend stops serving pages

EARLY WARNING SIGNS:
- Database active connections: Should spike 30s before pool exhaustion
  - Monitored? NO [MISSING OBSERVABILITY]
  - Alert configured? NO [MISSING OBSERVABILITY]

DESIGN DECISIONS THAT ACCELERATED:
- Connection pool has no max + no circuit breaker → once pool is exhausted, queued requests wait indefinitely rather than fail fast
- API timeout is 30s → long tail of slow requests accumulates → pool doesn't recover
```

#### Pre-Action Validation Checklist (Before Running This Card)

- [ ] **Architecture description includes all components** — If missing database, queue, or cache components, output will be vague
- [ ] **Team knows their SLA and incident response capability** — If you don't know acceptable downtime, this card's insights won't drive action
- [ ] **You've identified failure scenarios you care about** — This card works best if you already have 2–3 known risks
- [ ] **Monitoring/alerting is at least partially in place** — If you have zero observability, output will list [MISSING OBSERVABILITY] everywhere; that's OK, but have an action plan to add it

---

### Card 3: ARB Simulation Review

**When to Use:** Before a real Architecture Review Board meeting; or to pressure-test design against stakeholder concerns  
**When NOT to Use:** Don't use if you have actual ARB members available; hold a real review instead.  
**Phase:** Architecture Exploration → Adversarial Review  
**Risk Focus:** Stakeholder concerns, non-technical constraints, business tradeoffs

#### Prerequisites
- Architecture decision + summary of why you chose it
- Current constraints: team, budget, timeline, business goals
- Real stakeholder personas (or use defaults)

#### The Prompt

```text
You are an Architecture Review Board evaluating this decision.

Roles:
- CTO (cares about: technical risk, innovation fit, team scale-up)
- CISO (cares about: data flow, compliance, attack surface)
- VP Finance (cares about: cost, ROI, operational overhead)
- SRE Lead (cares about: operational feasibility, incident response, observability)
- Principal Architect (cares about: long-term maintainability, tech debt, refactoring constraints)

Architecture Decision:
[PASTE DECISION + RATIONALE HERE]

Constraints:
- Team size: [N] (current), [N] (projected in 12 months)
- Budget: [annual spend]
- Timeline to launch: [weeks/months]
- Business KPI: [e.g., "reduce customer churn by 20%"]
- Compliance/regulatory: [GDPR, HIPAA, PCI-DSS, or NONE]

Each role must:
1. Ask 2–3 HARD questions (not softball, genuinely challenging)
2. Identify 1–2 risks this decision does NOT address
3. Identify 1 assumption that, if false, breaks the decision
4. Propose 1 alternative (just one; not a full re-design)

RULES FOR EACH ROLE:
- CTO: Focus on whether this choice scales with team. Will we regret this in 12 months when team doubles?
- CISO: Map data flows. Where does sensitive data live? Who can access? What's blast radius if a component is compromised?
- Finance: Calculate total cost (infrastructure + operations labor). Is this cost justified by the business KPI?
- SRE: Do we have the operational maturity to run this? What's missing? What's the incident response playbook?
- Principal Architect: If we build this, what's the refactoring cost if business requirements change? Can we evolve it?

TONE:
- Be constructively critical, not dismissive
- Challenge the decision, not the team making it
- Propose concrete remediation, not vague concerns

OUTPUT FORMAT:

CTO:
- Hard Questions:
  1. [Question 1]
  2. [Question 2]
- Risks Not Addressed: [Risk 1], [Risk 2]
- Assumption at Risk: [Assumption: if false, breaks decision]
- Alternative to Consider: [Alternative and why]

CISO:
[Same format]

VP Finance:
[Same format]

SRE Lead:
[Same format]

Principal Architect:
[Same format]

SYNTHESIS (end of output):
- Decision Strengths: [List 2–3]
- Decision Weaknesses: [List 2–3]
- Critical Unknowns: [List 2–3 assumptions that need validation before go-live]
- Recommended Next Steps: [Specific actions to address weaknesses]
```

#### Example Output (Grounded)
```
CTO:
- Hard Questions:
  1. Our team is 6 people today. You're proposing microservices. In 12 months at current hiring pace, we'll be 12 people. Can we operate 8+ independent services with that team size? Or will we spend all our time on deployment pipelines?
  2. You're using a polyglot stack (Node + Python + Go). Do we have expertise in all three? If not, who's on-call for the Go service at 3 AM?
- Risks Not Addressed: Polyglot complexity; lack of shared logging/tracing setup
- Assumption at Risk: Assumes team willingness to adopt Kubernetes; if we don't, operational cost doubles
- Alternative: Monolith + separate background worker (Node + Node, simpler team scaling, easier on-call)

CISO:
- Hard Questions:
  1. You're storing customer PII in the customer service database. Who can query that database? Any service? Or is there access control?
  2. If the auth service is compromised, what's the blast radius? Can an attacker impersonate any user?
- Risks Not Addressed: No encryption-at-rest; no audit trail of who accessed PII
- Assumption at Risk: Assumes network is trusted; if internal network is compromised, all services are compromised
- Alternative: API gateway + strict access control + encryption-at-rest for PII

VP Finance:
- Hard Questions:
  1. Microservices means more servers, more observability infrastructure, more on-call burden. Total annual cost is $500K for infrastructure + $600K for ops labor = $1.1M. Our business KPI is worth $2M additional revenue. Break-even is 7 months. Do we have 7 months?
  2. If we hit a scaling wall in 6 months (earlier than expected), can we roll back to monolith? Or are we committed?
```

---

#### Pre-Action Validation Checklist (Before Running This Card)

- [ ] **You have 4–5 stakeholder roles represented or can name them** — ARB simulation works best with diverse perspectives
- [ ] **You know the decision or architecture being reviewed** — If the design is still vague, output will be generic objections
- [ ] **You're prepared for disagreement** — Each role raises real concerns; have space for debate before running this
- [ ] **You have a way to capture decisions** — Output will list hard questions; assign owners to answer each one

---

### Card 4: Assumption Attack Review

**When to Use:** After you've gathered requirements and drafted architecture; before detailed design  
**When NOT to Use:** Skip if your team has already validated assumptions in earlier sections.  
**Phase:** Requirements → Architecture Exploration  
**Risk Focus:** Unstated assumptions, hidden dependencies, misalignment

#### Prerequisites
- Architecture description with constraints
- Requirements document or decision rationale
- Team composition (skills, availability)

#### The Prompt

```text
Attack the ASSUMPTIONS in this architecture. Every design choice assumes something about:
- The environment (cloud, on-prem, hybrid?)
- The team (skills, size, availability?)
- The business (timeline, budget, competitive pressure?)
- The users (geography, scale, behavior?)
- Vendors and dependencies (capabilities, reliability, cost?)

Architecture:
[PASTE HERE]

Constraints:
[PASTE HERE]

For EACH component or major decision, answer:

1. WHAT IS THIS COMPONENT ASSUMING ABOUT THE TEAM?
   - Skills: Does the team have expertise? If not, are they learning it?
   - Availability: Is someone responsible? On-call? Training?
   - Cognitive load: Can a team member explain this component to someone else?
   
   For each skill gap, mark: [SKILL GAP: <skill>, currently <status>, risk if <unknown>]

2. WHAT IS THIS COMPONENT ASSUMING ABOUT THE ENVIRONMENT?
   - Latency: Between components, to cloud, to on-prem?
   - Network reliability: Automatic failover? Or manual?
   - Cost model: How much per GB of data? Per request? Per month?
   - Vendor lock-in: Can we move to a different cloud provider?
   
   For each environmental assumption, mark: [ENVIRONMENT ASSUMPTION: <assumption>, validated? yes/no]

3. WHAT IS THIS COMPONENT ASSUMING ABOUT BUSINESS CONSTRAINTS?
   - Timeline: Do we have time to build this complexity, or should we simplify?
   - Budget: Is the operational cost acceptable?
   - Competitive pressure: Do we need this level of sophistication to beat competitors? Or is it overengineering?
   
   For each business assumption, mark: [BUSINESS ASSUMPTION: <assumption>, discussed with stakeholders? yes/no]

4. WHAT IS THIS COMPONENT ASSUMING ABOUT USERS?
   - Geography: Are all users in one region, or globally distributed?
   - Scale: Peak concurrent users? Growth rate?
   - Behavior: Are they power users or casual? Do they expect sub-100ms latency or is 1s OK?
   
   For each user assumption, mark: [USER ASSUMPTION: <assumption>, data-driven? yes/no or <data source>]

OUTPUT FORMAT:

COMPONENT: [name]

TEAM ASSUMPTIONS:
- [SKILL GAP or ASSUMPTION: ...] (validated? yes/no)
- [...]

ENVIRONMENT ASSUMPTIONS:
- [ENVIRONMENT ASSUMPTION: ...] (validated? yes/no)
- [...]

BUSINESS ASSUMPTIONS:
- [BUSINESS ASSUMPTION: ...] (validated? yes/no)
- [...]

USER ASSUMPTIONS:
- [USER ASSUMPTION: ...] (data-driven? yes/no)
- [...]

SUMMARY (end of output):
- Critical Assumptions (if false, architecture fails): [List]
- Unvalidated Assumptions (need to confirm before build): [List]
- Risk Mitigations (how to reduce risk if assumption is wrong): [For each critical assumption, propose mitigation]
```

#### Example
```
COMPONENT: Real-Time Notification Service

TEAM ASSUMPTIONS:
- [SKILL GAP: WebSocket expertise. Currently: 0/6 engineers. Risk: Single point of failure; hard to debug; slow incident response]
- [SKILL GAP: DevOps for Kafka. Currently: 1/6 engineer (junior). Risk: If Kafka cluster fails, no one knows how to restore]

BUSINESS ASSUMPTIONS:
- [BUSINESS ASSUMPTION: Real-time notifications are worth 40% more operational complexity] (discussed with product? NO — [REQUIRES VALIDATION])
- [BUSINESS ASSUMPTION: Budget includes $200K/year for Kafka infrastructure] (discussed with finance? NO — [REQUIRES VALIDATION])

UNVALIDATED ASSUMPTIONS:
- Assumes team can add 2 more engineers skilled in Kafka/WebSockets in next 6 months
- Assumes AWS Lambda (backup notification service) can handle 50K notifications/second (not tested)
- Assumes mobile clients can handle connection drops every 30 seconds (not measured in user testing)
```

---

### Card 5: Dependency Risk Attack Review

**When to Use:** When architecture depends on external services, APIs, vendors, or multi-team handoffs  
**When NOT to Use:** Don't use for internal-only architectures with no external integrations.  
**Phase:** Architecture Exploration → Adversarial Review  
**Risk Focus:** Cascade failures, vendor lock-in, integration brittleness

#### Prerequisites
- Architecture with external dependencies marked
- SLA requirements
- Incident response expectations

#### The Prompt

```text
This architecture depends on external services. Attack those dependencies.

Dependencies:
[LIST EACH:
- Service name
- What breaks if it's unavailable
- SLA (availability %, latency p99)
- Fallback strategy (if any)
- Owner/vendor
]

For EACH external dependency, answer:

1. WHAT'S THE BLAST RADIUS IF THIS DEPENDENCY FAILS?
   - Which features stop working?
   - Are users partially impacted or completely blocked?
   - Can we degrade gracefully (serve stale data, reduced functionality)?
   - Or is failure total (all users see error)?
   
   Mark: [BLAST RADIUS: <scope>, graceful degradation? yes/no]

2. CAN WE DETECT THE FAILURE QUICKLY?
   - Do we have a health check? Explicit timeout?
   - Or is failure silent (slow responses masquerading as slow network)?
   - If we detect failure in 30 seconds, can we failover in <2 minutes?
   
   Mark: [DETECTION: <method>, MTTR: <estimated time>]

3. DO WE HAVE A FALLBACK?
   - Circuit breaker? Cache? Backup service?
   - If yes: How fresh is the fallback data? How long can we run on fallback?
   - If no: [VENDOR LOCK-IN RISK: Hard dependency with no fallback]

4. DOES THIS DEPENDENCY VIOLATE OUR SLA?
   - Our SLA says 99.9% availability (52.6 minutes downtime/year)
   - If dependency has 99% SLA (87.6 hours downtime/year), we violate our SLA unless we have redundancy
   - Calculate: If dependency is down for 1 hour, do we still meet SLA?

5. WHAT VENDOR LOCK-IN OR SWITCHING COSTS EXIST?
   - If we need to switch vendors (cost, competitive pressure, vendor bankruptcy), what's the effort?
   - Data migration effort? API rewrite? Downtime?
   - Is this acceptable or unacceptable risk?

OUTPUT FORMAT:

DEPENDENCY: [name]
- Owner: [vendor/team]
- Current SLA: [availability %]
- Our SLA requirement: [availability %]

BLAST RADIUS: [scope]; Graceful degradation: [yes/no]
DETECTION: [method]; MTTR: [estimated]
FALLBACK: [strategy or "NONE" + [VENDOR LOCK-IN RISK]]
SLA VIOLATION RISK: [if dependency fails 1 hour, do we violate SLA? yes/no]
SWITCHING COST: [effort to replace; acceptable risk? yes/no]

CRITICAL DEPENDENCIES (high blast radius + no fallback):
- [Dependency 1]
- [Dependency 2]

SUMMARY:
- Total blast radius: [what % of functionality is at risk if any dependency fails?]
- SLA-violating dependencies: [list]
- Vendor lock-in risks: [list]
- Recommended mitigations: [for each critical dependency]
```

#### Example
```
DEPENDENCY: Payment Gateway (Stripe)
- Current SLA: 99.9% (43 minutes downtime/month)
- Our SLA: 99.99% (4.3 minutes downtime/month)

BLAST RADIUS: All paid features blocked; users see error; no graceful degradation
DETECTION: Timeout on API call (30s); MTTR: ~2 minutes (manual failover)
FALLBACK: NONE [VENDOR LOCK-IN RISK: Hard dependency]
SLA VIOLATION RISK: YES — If Stripe is down for 1 hour, we violate SLA

RECOMMENDED MITIGATIONS:
- Add payment-retry queue + offline payment initiation
- Add circuit breaker: if Stripe unavailable >1 min, queue payment + return "processing" to user
- Test failover monthly
```

---

#### Pre-Action Validation Checklist (Before Running This Card)

- [ ] **You have a list of external dependencies** — If not obvious, spend 10 minutes drawing dependency diagram first
- [ ] **You know your team's operational capability** — If you don't have circuits breakers/monitoring/fallbacks today, this card will surface missing pieces
- [ ] **You understand SLA math** — Dependency SLA × Your SLA means your effective SLA is lower
- [ ] **You're ready for "no fallback" findings** — If dependencies have no fallback, this card will flag [VENDOR LOCK-IN RISK]; have a decision plan

---

### Card 6: Operability Attack Review

**When to Use:** After architecture is drafted; before hand-off to operations/SRE  
**When NOT to Use:** Skip if operations are handled by a specialized team with documented runbooks.  
**Phase:** Architecture → Documentation/Operations Planning  
**Risk Focus:** Operational readiness, runbook feasibility, incident response

#### Prerequisites
- Architecture description
- Operational team size and skill level
- Current monitoring/observability setup
- Incident response time SLA

#### The Prompt

```text
Your ops team has to RUN this architecture 24/7/365. Attack the operational assumptions.

Operational Context:
- Team size: [N people]
- On-call rotation: [how many rotations/depth?]
- Current monitoring stack: [Prometheus, DataDog, CloudWatch, custom, other?]
- Incident response SLA: [minutes to first action, hours to resolution]
- Existing runbooks for: [list what they already have playbooks for]

Architecture:
[PASTE HERE]

For EACH component, answer:

1. WHAT MUST THE OPS TEAM KNOW TO OPERATE THIS SAFELY?
   - Normal behavior: What should this component's metrics look like? (What's "healthy"?)
   - Warning signs: What metric spikes 10 minutes before failure?
   - Failure mode: How does failure manifest? (Error rate spikes? No traffic? Silent failures?)
   - Action: If the warning sign appears, what should ops do?
   
   Mark each: [RUNBOOK MISSING: <component>] or [RUNBOOK EXISTS: <location>]

2. WHAT OBSERVABILITY IS REQUIRED?
   - Metrics needed: [CPU, memory, latency p50/p99, error rate, queue depth, etc.]
   - Logs needed: [error logs? trace logs? audit logs?]
   - Traces needed: [are distributed traces required to debug failures?]
   - Dashboards: Is there a dashboard showing health of this component?
   
   For each missing observability, mark: [MISSING OBSERVABILITY: <metric>]

3. WHAT COULD GO WRONG THAT OPS WOULD NOT DETECT?
   - Silent failures: No error, no metrics spike, but system is broken (e.g., cache not invalidating)
   - Slow failures: Metrics degrade gradually; ops might not notice until SLA is violated
   - Cascading failures: Failure in component A doesn't show up as failure in component A; shows up as timeout in component B
   
   For each, mark: [UNDETECTABLE FAILURE MODE: <description>]

4. HOW LONG TO MANUALLY REMEDIATE?
   - If on-call engineer is paged at 3 AM: How long to diagnose? To fix? To verify?
   - Timeline: Page → Log in → Diagnose (how long?) → Escalate or fix (how long?)
   - Can this be done in [incident response SLA] or does it exceed it?
   
   If remediation time > SLA: [OPERABILITY VIOLATION: Manual remediation exceeds SLA]

5. WHAT WOULD AUTOMATION HELP?
   - Auto-scaling? Auto-failover? Auto-healing? Auto-rollback?
   - Which of these are already built into the architecture?
   - Which are missing and should be added?
   
   For each gap: [AUTOMATION GAP: <capability>, required? yes/no, effort to add? low/medium/high]

OUTPUT FORMAT:

COMPONENT: [name]

RUNBOOK STATUS:
- [RUNBOOK EXISTS: <location>] OR [RUNBOOK MISSING: required before launch]

OBSERVABILITY:
- Metrics: [list required]
- Logs: [list required]
- Traces: [yes/no]
- Dashboard: [exists/missing]
- [MISSING OBSERVABILITY: <metric>] (required before launch)

UNDETECTABLE FAILURE MODES:
- [Description 1] (likelihood: [low/medium/high])
- [Description 2]

MANUAL REMEDIATION TIME:
- Estimated diagnosis + fix time: [X minutes]
- Incident response SLA: [Y minutes]
- Violates SLA? [yes/no]
- [If yes: OPERABILITY VIOLATION: Requires automation or SLA revision]

AUTOMATION GAPS:
- [Auto-scaling status]
- [Auto-failover status]
- [AUTOMATION GAP: <capability>, effort: <effort>]

SUMMARY (end of output):
- Operational Readiness: [Ready / Needs Work / Cannot Operate Without Changes]
- Critical Gaps (must fix before go-live): [List]
- Should-Fix Gaps (address post-launch): [List]
```

#### Example
```
COMPONENT: Message Queue

RUNBOOK STATUS: [RUNBOOK MISSING: How to drain backlog in emergency]

OBSERVABILITY:
- Metrics: Queue depth, message lag, consumer lag
- [MISSING OBSERVABILITY: Consumer group status dashboard]
- Dashboard: Exists in DataDog but not linked in team wiki

UNDETECTABLE FAILURE MODES:
- Queue processing "silently slow": Messages are being processed but at 1/10 speed. Ops won't see error until customer complaints.

MANUAL REMEDIATION TIME:
- Diagnosis (find that queue is slow): 5–10 min
- Fix (restart consumers? Adjust parallelism?): 5–20 min
- Total: 10–30 minutes
- SLA: 5 minutes to mitigation
- [OPERABILITY VIOLATION: Manual remediation exceeds SLA; requires auto-scaling or circuit breaker]

AUTOMATION GAPS:
- [Auto-scaling: Missing; effort: medium; required before launch]
- [Alert on queue lag > 10 minutes: Missing; effort: low; required before launch]
```

---

#### Pre-Action Validation Checklist (Before Running This Card)

- [ ] **You know your operational team's current capability** — If you haven't done incident response before, this card will surface gaps
- [ ] **You have at least a basic monitoring/alerting setup** — If you have zero observability, the output will be mostly [MISSING OBSERVABILITY]; that's OK, use it to justify building observability
- [ ] **You're prepared for "manual remediation time > SLA" findings** — This card often surfaces operability violations; have a backlog to fix them
- [ ] **You understand runbook scope** — Runbooks are operational procedures (not architecture changes); if gaps exist, they're feasible to fix

---

## How to Use

### Full Adversarial Pass (Comprehensive Review)

1. **Adversarial Architecture Review** — high-level gaps
   - Input: Architecture summary + constraints
   - Output: List of hidden assumptions, risks by category

2. **Scaling & Performance Attack** — what breaks under load?
   - Input: Component list + expected load profiles
   - Output: Bottlenecks, caching gaps, data hotspots

3. **Security & Data Flow Review** — who can access what?
   - Input: Data classification + access patterns
   - Output: Security gaps, blast radius per component, data flow violations

4. **Operational Readiness Challenge** — can you actually run this?
   - Input: Architecture + operational team size/maturity
   - Output: Observability gaps, runbook gaps, skill requirements

5. **Catastrophic Failure Analysis** — assume it failed in production
   - Input: Your architecture
   - Output: Most likely failure chains, early warning signs missed, design contributors

6. **ARB Simulation** — skeptical review board
   - Input: Architecture + decision summary
   - Output: Hard questions from CTO, CISO, SRE, Finance perspectives

### Quick Adversarial Pass (30 minutes)

- Run **Adversarial Architecture Review** (high-level)
- Run **ARB Simulation** for your organizational context (skip generic personas if you have a real ARB coming)
- Skip detailed scaling/security/operational deep dives unless the high-level review surfaces specific risks

### Targeted Reviews (Risk-Specific)

- **If performance is critical:** Run Scaling & Performance Attack first
- **If security is critical:** Run Security & Data Flow Review first
- **If operational complexity is high:** Run Operational Readiness Challenge first

---

## Customization Guidance

### By Architecture Style

- **Monolith:** Focus on operational readiness + scaling bottlenecks. Security review is simpler (fewer trust boundaries).
- **Microservices:** Focus on integration risks + distributed failure chains. Add data consistency review.
- **Event-Driven:** Focus on ordering guarantees + message loss scenarios. Add operational complexity review.
- **Serverless:** Focus on cold-start impact + cost surprises. Add vendor lock-in review.

### By Domain

- **High-frequency trading:** Scaling attack should focus on latency tail (p99, p999) not just throughput
- **Healthcare/Finance:** Security review should map to compliance requirements (HIPAA, PCI-DSS, SOX)
- **IoT/Embedded:** Operational readiness should focus on offline scenarios and network unreliability

### By Operational Maturity

- **Startup (no SRE):** Operational Readiness Challenge is highest priority (will catch unfeasible designs)
- **Mid-scale:** Focus on Scaling and Security reviews (operational maturity is emerging)
- **Enterprise:** All reviews equally important; use ARB Simulation with real stakeholder personas

---

## Quality Checks

### Before Using Review Output

- [ ] Risks are **grounded in stated constraints**, not invented (e.g., "10x scale will break caching" should reference your latency target or current load)
- [ ] Assumptions are **explicit and numbered** (e.g., "Assumes SRE team has 2+ years with Kubernetes" or "Assumes <100ms network latency")
- [ ] Failure modes describe **specific failure chains**, not generic risks (good: "If primary DB fails, replica becomes read-only but queue backs up → eventual memory exhaustion"; bad: "DB failure is risky")
- [ ] Security gaps map to **specific data assets or trust boundaries**, not vague threats (e.g., "API server can read customer PII; if compromised, blast radius includes all users' financial data")

### Red Flags (Hallucination Indicators)

- ❌ Generic risks appear regardless of architecture ("scalability is hard" applies to everything) — should be *specific* to your design
- ❌ Catastrophic failure analysis invents unrealistic scenarios ("cosmic rays flip a bit") — base on known failure patterns and real case studies
- ❌ ARB personas suggest solutions instead of asking questions (out of scope; should challenge, not design)
- ❌ Assumptions aren't stated; instead output feels confident but groundless — mark as `[REQUIRES VALIDATION]`

### Success Indicators

- ✅ Review surfaces 3+ assumptions you didn't realize you were making
- ✅ Failure chain analysis describes *how* cascading failures happen, not just that they *can*
- ✅ Security review maps to your data classification (not generic "data security is important")
- ✅ Operational challenge surfaces actual gaps (e.g., "no alerting strategy for message queue depth" vs. generic "observability is hard")

---

## Next Steps

After adversarial review:
1. **Document findings:** Create a Risk Register (template in `/shared/risk-document.instructions.md`)
2. **Validate assumptions:** Update your Assumption Register with all stated assumptions
3. **Make decisions:** Move to **04 — Architecture Documentation** to record how you addressed each risk
4. **Run through ARB:** If you have a real Architecture Review Board, use the review findings as input to their meeting

---

## Resources

- Shared personas: `/shared/personas.md` (for ARB simulation)
- Risk document template: `/shared/risk-document.instructions.md`
- Assumption register template: `/shared/assumption.instructions.md`
- Known failure patterns: `/shared/failure-patterns.md` (use as reference for Catastrophic Failure Analysis)
- Anti-slop checklist: `/shared/anti-slop-checklist.md` (use to catch invented risks vs. real ones)

---

**Status:** Complete (6 prompt cards, guardrails applied, anti-hallucination markers, validation gates)
**Last Updated:** 2026-06-01T21:32:47Z (Morpheus — Prompt Card Implementation)
