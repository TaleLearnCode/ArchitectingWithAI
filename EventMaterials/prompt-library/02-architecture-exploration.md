# 02 — Architecture Exploration Prompts

*Widen the solution space, force constraint-driven filtering, and justify tradeoffs.*

---

## Section Purpose

Once requirements are clear, the solution space is vast. Picking the first architecture that comes to mind is how architects miss better options — or lock into a pattern that doesn't fit the constraints.

These prompts help you:
- **Generate multiple architecture approaches** that fit the stated constraints
- **Compare options across quality attributes** (scalability, cost, operational complexity, security, etc.)
- **Justify tradeoffs explicitly** — not with generic 1–5 ratings, but with constraint-grounded reasoning
- **Identify hidden risks** in each option (team skill gaps, operational maturity required, failure modes)
- **Eliminate architectures** that violate stated constraints early

**Thesis Connection:** AI broadens exploration. Here it generates multiple architectures and exposes their tradeoffs so architects can supply judgment about which fits.

---

## Prompt Catalog

## ✓ Prompt 1: Constraint-Driven Architecture Generation

### Use This When
You have clear functional requirements and constraints (team size, budget, latency, deployment environment). You want to generate 3–5 distinct architecture approaches that explicitly respect your constraints, not generic templates.

### When NOT to Use
Skip if constraints are vague or shifting; use Section 01 first to clarify.

### Prerequisites
- Functional and non-functional requirements (from Section 1)
- Constraints inventory: team size, deployment environment, latency/performance targets, budget, regulatory constraints, operational maturity
- Project domain (e.g., SaaS, real-time analytics, batch processing)
- Technology preferences or restrictions (if any)

**Validation Rule:** If constraints are missing, output will say `[INPUT MISSING: <constraint name>]`. Do not proceed with assumptions.

---

### Input Example

```
PROJECT CONTEXT:
Domain: E-commerce platform
Requirements: Product catalog (1M+ items), search < 500ms p99, real-time inventory, 
payment processing, GDPR/PCI-DSS compliance

CONSTRAINTS:
- Team: 8 engineers (2 frontend, 4 backend, 1 SRE, 1 QA)
- Deployment: AWS, containerized
- Budget: $150K/year infrastructure
- Time-to-market: 6 months
- Operational maturity: Intermediate (CI/CD in place, basic monitoring)
- Latency target: <500ms p99 for search, <100ms p99 for product detail
- Data residency: EU only (GDPR)
- Scaling: Expect 10–100K concurrent users over 18 months
```

---

### The Prompt

```text
You are an architecture advisor. Generate 4 distinct architecture approaches that solve this problem 
while respecting ALL stated constraints.

For each architecture, provide:

1. NAME & OVERVIEW (1–2 sentences)
2. COMPONENT BREAKDOWN (key components, responsibilities, how they communicate)
3. CONSTRAINT COMPLIANCE (check each constraint; mark as FITS, VIOLATES, or REQUIRES VALIDATION)
4. TEAM FIT (how many engineers needed per role; required expertise; learning curve)
5. OPERATIONAL PROFILE (deployment complexity, observability requirements, failure modes)
6. COST ESTIMATE (rough infrastructure cost; operational cost; total first-year cost)
7. HIDDEN RISKS (team skill gaps, technology immaturity, failure modes if constraint assumptions change)
8. WHEN TO CHOOSE THIS (specific project characteristics that make this ideal)

CRITICAL RULES:
- Do NOT include generic templates. Each option must respect the stated constraints.
- If an architecture violates a constraint, say so explicitly (do not suggest a workaround).
- Cost estimates must reference specific AWS services or equivalent.
- Team fit must assume the stated team size and maturity level.
- Do NOT invent new technologies or capabilities not mentioned in the constraints.

***
[PASTE PROJECT CONTEXT AND CONSTRAINTS HERE]
```

---

### Customization Knobs

| Knob | Use When | Example |
|------|----------|---------|
| **Depth** | You want detailed component interactions | Add: "For each component, describe synchronous vs. asynchronous communication patterns" |
| **Cost focus** | Budget is the primary constraint | Add: "Optimize for lowest first-year infrastructure cost; assume managed services preferred" |
| **Team maturity** | Your team is less experienced | Add: "Assume no Kubernetes expertise; favor managed platforms (serverless, App Engine)" |
| **Regulatory focus** | Compliance is critical | Add: "For each architecture, explain how it meets GDPR, PCI-DSS, and audit requirements" |
| **Speed focus** | Time-to-market dominates | Add: "Rank options by time-to-MVP; assume 6-month constraint is hard" |

---

### Example Input & Output

```
INPUT (attendee provides):
Team: 3 backend engineers, 1 DevOps, 1 QA | Budget: $50K/year | Latency: <100ms p99 | 
Deployment: AWS | Users: 100 → 10K over 12 months | Regulatory: SOC2

OUTPUT (AI provides):

OPTION 1: Monolith + Managed Services
OVERVIEW: Single Node.js application deployed to AWS App Runner, PostgreSQL on RDS, 
Redis for cache, S3 for file storage.

COMPONENT BREAKDOWN:
- API Server (1 App Runner service, auto-scaling 1–20 instances)
- PostgreSQL (RDS Multi-AZ for high availability)
- Redis (ElastiCache for session + cache)
- S3 (file storage, CloudFront CDN)
- CloudWatch + Alarms (monitoring)

CONSTRAINT COMPLIANCE:
- Team (3 backend): ✅ FITS — Single monolith is straightforward for small team
- Budget: ✅ FITS — ~$40K/year (App Runner $2K/month, RDS $1K, Redis $300, S3/CDN $200)
- Latency <100ms: ✅ FITS — Single datacenter, optimized DB queries, Redis caching
- AWS: ✅ FITS — All managed services
- Scale 100→10K: ✓ FITS with planning — App Runner auto-scales; DB needs optimization at 5K+ users
- SOC2: ✅ FITS — Use RDS encryption, VPC isolation, CloudTrail logging

TEAM FIT: 1 engineer per 2–3 core features; 1 DevOps manages deployment/monitoring; 
learning curve low (Node + PostgreSQL common skills).

OPERATIONAL PROFILE: Deployment via git push (GitHub Actions → App Runner); 
vertical scaling handled automatically; monitoring via CloudWatch. Database optimization 
required at 5K+ users (add read replicas, query indexing).

COST ESTIMATE: Infrastructure $40K/year (App Runner $24K, RDS $12K, Redis $3.6K, CDN $0.4K).
Operations: 1 DevOps full-time.

HIDDEN RISKS:
- At 5K+ users, single monolith may become coordination bottleneck for parallel development
- Database write bottleneck if product updates spike
- Team has no multi-region experience; disaster recovery untested
- Compliance: SOC2 requires audit trail; implement now or migrate later (expensive)

WHEN TO CHOOSE: Small, experienced team; 12-month runway; willing to monolith first, 
microservices later if needed.
```

---

### Example: Weak Output (and How to Remediate)

```
INPUT (Same context as strong example):
Team: 3 backend engineers, 1 DevOps, 1 QA | Budget: $50K/year | Latency: <100ms p99 | 
Deployment: AWS | Users: 100 → 10K over 12 months | Regulatory: SOC2

WEAK OUTPUT (Generic template, ignoring constraints):

OPTION 1: Standard Monolith
OVERVIEW: Traditional three-tier architecture (web + application + database).

COMPONENT BREAKDOWN:
- Web Server (Apache)
- Application Server (Java Spring Boot)
- Database (PostgreSQL)

CONSTRAINT COMPLIANCE:
- Team: ✅ Generally good for small teams
- Budget: ? Unclear (no cost estimates)
- Latency: ? Should be fine
- AWS: ? Not mentioned
- Scale 100→10K: ✓ Monoliths scale reasonably
- SOC2: ✓ Standard approach

TEAM FIT: Any team can build a monolith.

HIDDEN RISKS: None identified.

WHEN TO CHOOSE: Always a safe default.

---

OPTION 2: Microservices
OVERVIEW: Decompose into independent services.

[Vague benefits, no constraint analysis]

---

WHY THIS IS WEAK:
1. ❌ Cost estimates missing — No mention of actual AWS services or pricing
2. ❌ Latency assumption unjustified — "Should be fine" with no analysis
3. ❌ Team fit is generic — Any team can build a monolith (but can THIS team?)
4. ❌ Constraint compliance too vague — No explicit FITS/VIOLATES/VALIDATION for each constraint
5. ❌ Deployment environment ignored — AWS mentioned in constraint but not addressed in architecture
6. ❌ Hidden risks not identified — "None identified" for a production system is a red flag
7. ❌ Generic templates — Could apply to any project; not constraint-driven
8. ❌ Only 2 options — Should generate 3–5 distinct approaches per constraint

---

REMEDIATION:

**Requirement:** Generate constraint-driven architectures, not generic templates.

**Actions:**
1. For each constraint (team: 3 backend), explicitly check FITS/VIOLATES/NEEDS VALIDATION
2. Cost estimate: Reference actual AWS services (App Runner vs. EC2 vs. ECS) and pricing per month
3. Latency analysis: Map each architecture's communication pattern to latency target (<100ms p99)
4. Deployment: Specifically address AWS managed services (not "deployment environment AWS")
5. Hidden risks: Name specific failure modes for this team + budget (e.g., "At 5K users, single monolith database becomes bottleneck because we have no DBA for optimization")
6. Generate 3–5 distinct options (monolith, microservices, serverless, event-driven, etc.) and explain WHY each respects or violates constraints
7. For scale 100→10K: Be specific about scaling limits (e.g., "Monolith scales to 5K users before DB writes saturate; at 10K we need sharding or read replicas")

**Regenerate with:**
- 4 distinct architecture options
- Explicit constraint compliance (FITS/VIOLATES/NEEDS VALIDATION per constraint)
- Cost breakdown by service (App Runner $X/month, RDS $Y, etc.)
- Latency analysis specific to this architecture (e.g., "no network hops" for monolith = <50ms DB roundtrip)
- Team fit specific to "3 backend engineers" (not generic "small teams")
- Named risks tied to this project (e.g., "SOC2 audit trail requires immutable logging; not built into monolith by default")
```

---

### Validation Checklist

- [ ] **Constraint compliance is explicit** — Each constraint marked FITS/VIOLATES/VALIDATION NEEDED (not vague)
- [ ] **Cost estimates are grounded** — References specific services and assumptions (e.g., "RDS Multi-AZ at db.t4g.medium")
- [ ] **Team fit is realistic** — Acknowledges team size and experience, not invented headcount
- [ ] **Risk section names specific gaps** — Not just "may need optimization" but "database write bottleneck at 5K users"
- [ ] **No generic templates** — Each option unique to this project's constraints
- [ ] **Hidden risks are testable** — Claims about failure modes can be validated (not speculation)

---

### Pro Tips

- **Timing:** Run this after Section 1 requirements are finalized. If constraints are changing, loop back to Section 1.
- **Team collaboration:** Share results with tech lead + DevOps. Have them validate team fit and cost estimates.
- **Common mistakes:** Architects often pick architectures that match their experience, not the project constraints. Use constraint compliance checklist to filter.
- **Scaling:** If you have 10+ constraints, group them by category (team, deployment, regulatory, performance) and validate each group separately.

---

### Guardrails Applied

- **Pattern:** Prevents "generic monolith vs. microservices" boilerplate
- **How it prevents:** Requires explicit constraint compliance check for every option; any violated constraint triggers escalation
- **Example failure if guardrail removed:** Output becomes "Use microservices for scalability, monolith for simplicity" (generic, context-free)

---

### Related Prompts

- **Use together with:** Prompt 2 (Tradeoff Analysis) — compare options across multiple quality attributes
- **Use instead of:** Generic "4 architecture patterns" templates
- **Next step:** Prompt 5 (Risk Analysis per Option) — deep dive on hidden risks in shortlisted options

---

## ✓ Prompt 2: Constraint-Grounded Tradeoff Analysis

### Use This When
You have 3–5 architecture options from Prompt 1, and you need to compare them objectively without generic 1–5 ratings. You want to see how each constraint impacts each option.

### When NOT to Use
Don't use for binary decisions (go/no-go); this requires multiple options to compare.

### Prerequisites
- 3–5 architecture options (from Prompt 1, or manually defined)
- Constraint list with explicit values (not vague: "low cost" → "$50K/year max")
- Quality attributes you care about (performance, reliability, cost, team velocity, etc.)
- Project timeline and scaling expectations

**Validation Rule:** If constraints lack explicit values, output will say `[INPUT MISSING: Explicit Constraint Values]`. Replace vague constraints (e.g., "low cost") with numbers (e.g., "$50K/year max"). If quality attributes are missing, you'll get generic comparisons; that's acceptable—use the output to drive discussion on what matters most.

### Input Example

```
ARCHITECTURES TO COMPARE:
1. Monolith + Managed Services
2. Microservices (3 services: API, Search, Payments)
3. Event-driven + Message Queue

CONSTRAINTS:
- Budget: $50K/year max
- Team: 3 backend, 1 DevOps
- Latency: <100ms p99 for search
- Scale: 100 → 10K users over 12 months
- Regulatory: SOC2

QUALITY ATTRIBUTES:
- Cost (infrastructure + ops)
- Deployment speed (can we ship features weekly?)
- Reliability (uptime requirement: 99.5%)
- Team velocity (how fast can 4 engineers add features?)
```

---

### The Prompt

```text
Create a tradeoff table comparing these architectures. For EACH cell:

1. Provide a METRIC (specific, measurable number—not a vague rating)
2. Explain the CONSTRAINT IMPACT (which constraint makes this metric matter?)
3. Justify the REASONING (why does Architecture A have this value and Architecture B different?)

Column headers:
- Architecture Name
- [Quality Attribute 1]: Metric (Reasoning)
- [Quality Attribute 2]: Metric (Reasoning)
- [Quality Attribute 3]: Metric (Reasoning)
- Constraint Violations (if any)
- Recommendation for This Project

Row headers:
- [Each architecture option]

CRITICAL RULES:
- NO 1–5 RATINGS. Every cell must have a measurable metric (cost in $/year, latency in ms, deployment time in minutes, etc.)
- EVERY metric must cite which constraint makes it matter
- If a metric depends on team behavior (not architecture), flag it as [TEAM DEPENDENT]
- If a metric is uncertain, mark it [ASSUMPTION: <description>] and explain why
- DO NOT invent metrics. Use standard measures: cost, latency, throughput, MTTR, incident count, feature velocity, etc.

***
[PASTE ARCHITECTURES, CONSTRAINTS, AND QUALITY ATTRIBUTES HERE]
```

---

### Customization Knobs

| Knob | Use When | Example |
|------|----------|---------|
| **Detail level** | You want deep cost breakdowns | Add: "For cost, break down by component (compute, database, networking, monitoring, personnel)" |
| **Risk focus** | You want to see failure modes | Add: "Add column: Failure modes under each constraint (what breaks first if constraint changes?)" |
| **Team velocity** | Shipping speed dominates | Add: "Add column: Time to add a new feature in each architecture (assuming 3 devs)" |
| **Scaling scenarios** | You want to see how options handle growth | Add: "For each metric, show the value at 1K users and again at 10K users" |

---

### Example Input & Output

```
INPUT:
Architectures: Monolith, Microservices (3 services), Event-driven
Constraints: $50K/year, 4-person team, <100ms p99 latency, 99.5% uptime

OUTPUT:

| Architecture | Cost (First Year) | Cost (Rationale) | Latency p99 | Latency (Rationale) | Deployment Time | Deployment (Rationale) | Constraint Violations |
|---|---|---|---|---|---|---|---|
| **Monolith + Managed** | $40K | App Runner $24K, RDS $12K, caching $3.6K; no personnel cost (existing ops team) | <100ms | Single DB roundtrip + Redis cache; no network hops between services | 5 min | git push → GitHub Actions → App Runner auto-deploy | None |
| **Microservices (3)** | $65K | Each service: App Runner $8K (3×$24K=$72K total compute overage), RDS reads replicas +$5K, inter-service network +$1K, monitoring agents +$2K | <200ms (p99) | Service-to-service RPC adds 50–100ms; search service calls API service adds latency | 30 min | Coordinate deployment across 3 services; test service boundaries | **VIOLATES: Budget** (+$15K); **VIOLATES: Latency** (>100ms); Team deploys less frequently to avoid cascading failures |
| **Event-Driven** | $55K | Message queue (SQS) $2K, topic-based routing $1K, additional monitoring $3K, dead letter queues $1K | <150ms (p99) | Asynchronous processing adds latency for consistency checks; strong consistency scenarios require synchronous fallback | 15 min | Simpler per-service deployment; eventual consistency reduces coordination | **ASSUMPTION: Team has event-driven expertise** [Team has not done this before; learning curve 4–6 weeks] |

WINNING OPTION FOR THIS PROJECT: Monolith + Managed Services
- Fits budget constraint
- Meets latency constraint
- Matches team size and experience
- Deployment is straightforward
- Clear upgrade path if architecture needs to change (add event queue later)
```

---

### Example: Weak Output (and How to Remediate)

```
INPUT (Same context):
Architectures: Monolith, Microservices (3 services), Event-driven
Constraints: $50K/year, 4-person team, <100ms p99 latency, 99.5% uptime
Quality attributes: Cost, Latency, Deployment speed, Reliability

WEAK OUTPUT (Generic 1-5 ratings, no constraint grounding):

| Architecture | Cost | Cost Rationale | Latency | Latency Rationale | Deployment | Deployment Rationale | Violations |
|---|---|---|---|---|---|---|---|
| **Monolith** | 3/5 | Reasonable cost | 4/5 | Generally fast | 4/5 | Simple deployments | None |
| **Microservices** | 2/5 | More expensive due to complexity | 2/5 | Inter-service communication adds overhead | 1/5 | Harder to deploy | Budget violation; latency may spike |
| **Event-driven** | 3/5 | Queue service costs moderate | 3/5 | Async adds some latency | 3/5 | Middle ground | Requires messaging expertise |

RECOMMENDATION: Monolith looks best based on ratings.

---

WHY THIS IS WEAK:
1. ❌ 1–5 ratings without metrics — "Cost 3/5" is meaningless; should be "$40K/year"
2. ❌ No constraint grounding — Doesn't explain which constraint makes each rating matter
3. ❌ Vague rationale — "Generally fast" is not analysis
4. ❌ "Budget violation" flagged but not quantified — How much over? Why?
5. ❌ Expertise mentioned but not tied to team constraints — "Requires messaging expertise" but we said 4-person team with unknown skills
6. ❌ Recommendation weak — "Looks best based on ratings" doesn't justify trade-offs
7. ❌ No [ASSUMPTION] tags — If recommendation depends on unvalidated factors, should be flagged

---

REMEDIATION:

**Requirement:** Every metric must be specific (dollar amount, ms, minutes) and tied to constraint impact.

**Actions:**
1. Replace all 1–5 ratings with NUMERIC METRICS:
   - Cost: $40K/year (not "3/5")
   - Latency: 120ms p99 (not "2/5")
   - Deployment time: 30 minutes (not "1/5")
2. Add [GROUNDED: <constraint>] tag explaining why this metric matters:
   - Cost $50K limit: [GROUNDED: stated budget constraint]
   - Latency <100ms: [GROUNDED: stated p99 target]
3. For each cell, explain the REASONING specific to this architecture:
   - Monolith latency: "Single database; no inter-service RPC; 50ms DB roundtrip + 30ms network = 80ms < 100ms target"
   - Microservices latency: "API → Search service RPC (50ms) + Search → DB (50ms) + network hops (20ms) = 120ms > 100ms target (VIOLATES)"
4. Quantify violations with fallback costs:
   - Microservices cost: "$65K (+ $15K over budget)" and "Can reduce by 30% if we use spot instances (risk: availability)"
5. For [ASSUMPTION] tags, name the assumption and confidence:
   - "Monolith OK to 5K users" [ASSUMPTION: no DBA on team; if DB tuning needed, may exceed 5K; confidence: 2/5]
6. Add team-specific analysis:
   - "Event-driven requires 4+ weeks learning curve for 4-person team (month 1 of 6-month project)"

**Regenerate with:**
- All metrics in specific units ($/year, ms, minutes, %)
- Every metric tagged with constraint impact [GROUNDED: ...]
- Reasoning specific to this architecture (not generic)
- Violations quantified (not just "violates")
- Assumptions named + confidence scored
- Recommendation includes contingency (e.g., "Monolith recommended; if scale hits 5K users before month 4, migrate plan ready by month 5")
```

---

### Validation Checklist

- [ ] **Every metric is measurable** (not "high," "scalable," "fast")
- [ ] **Metrics reference constraints** (e.g., "Cost must be <$50K" is explained in "Cost Rationale")
- [ ] **Assumptions are flagged** (if metric depends on unstated factors, marked [ASSUMPTION: ...])
- [ ] **Violations are explicit** (if architecture violates a constraint, highlighted in "Constraint Violations" column)
- [ ] **No invented metrics** (uses standard measures: $, ms, %, count, etc.)

---

### Pro Tips

- **Timing:** Run after Prompt 1. If tradeoff analysis shows all options violate constraints, loop back to requirements team.
- **Team collaboration:** Present tradeoff table to product + engineering leads. Have them validate cost estimates and team velocity assumptions.
- **Common mistakes:** Architects often use vague ratings ("scalable," "enterprise-grade") because concrete metrics are hard. Push for specifics or mark [SPECULATION].

---

## ✓ Prompt 3: Quality Attribute Scenarios (ISO 25010)

### Use This When
You have 2–3 shortlisted architecture options and want to stress-test them against non-functional requirements. You want to see how each option handles performance peaks, failures, security threats, and operational changes.

### When NOT to Use
Skip if your project has no explicit quality attribute targets.

### Prerequisites
- One architecture option (from Prompt 1 or Prompt 2)
- NFR targets (latency, throughput, availability, scalability, security sensitivity)
- Domain context (e.g., e-commerce needs high availability for checkout; analytics can tolerate delays)
- Failure scenarios you care about (database failure, service outage, traffic spike, etc.)

**Validation Rule:** If NFR targets are missing, output will say `[INPUT MISSING: NFR Targets]`. Provide numeric targets (e.g., "<100ms p99", "99.5% availability"). If failure scenarios are not defined, you'll get generic scenarios; that's acceptable—use the output to discuss which scenarios matter most.

### Input Example

```
ARCHITECTURE: Monolith + Managed Services (App Runner + RDS + Redis + S3)

NFR TARGETS:
- Latency (p99): <100ms
- Throughput: 1K requests/sec at peak
- Availability: 99.5% (4.38 hours downtime/month acceptable)
- Data consistency: Strong for payments, eventual for search cache
- Security: PCI-DSS for payment data, SOC2 for access logs

OPERATIONAL CONSTRAINTS:
- Team can be paged 24/7
- Team has basic Kubernetes experience (not needed here)
- No dedicated DBA
```

---

### The Prompt

```text
Generate Quality Attribute Scenarios (QAS) for this architecture using the ISO 25010 model.

For each scenario, provide:

1. STIMULUS (what event or action triggers this scenario?)
2. ENVIRONMENT (system conditions when stimulus occurs)
3. RESPONSE (what should the system do?)
4. RESPONSE MEASURE (how do we know it worked? Specific threshold, not vague)
5. EXPECTED RESULT (what actually happens in this architecture?)
6. PASS/FAIL (does this architecture meet the requirement?)
7. MITIGATION (if FAIL, what changes to make it PASS?)

Generate scenarios covering:
- PERFORMANCE (peak load, sustained load, degraded performance)
- RELIABILITY (component failure, cascading failures, recovery time)
- SECURITY (data breach attempt, DDoS, compromised credentials)
- USABILITY (error handling, user-visible failures)
- MAINTAINABILITY (code deploy, configuration change, emergency rollback)

CRITICAL RULES:
- Each scenario must be TESTABLE (you can write a test to verify it)
- Response measures must be MEASURABLE (milliseconds, percentage, count—not "fast," "reliable")
- Expected results must be SPECIFIC to this architecture (not generic)
- Do NOT invent capabilities the architecture doesn't have

***
[PASTE ARCHITECTURE AND NFR TARGETS HERE]
```

---

### Customization Knobs

| Knob | Use When | Example |
|------|----------|---------|
| **Stress scenarios** | You want extreme failure modes | Add: "Include: simultaneous database failure + traffic spike + deployment in progress" |
| **Cost-of-failure** | Business impact matters | Add: "For each failure scenario, estimate revenue impact (lost transactions, SLA penalties)" |
| **Operational maturity** | Your team is less experienced | Add: "Assume no on-call rotation; team is 9–5. How does MTTR change?" |
| **Compliance** | Regulatory scenarios | Add: "Include: audit log completeness during failure, data integrity after partial outage" |

---

### Example Input & Output

```
INPUT:
Architecture: Monolith + Managed Services
NFR: <100ms p99 latency, 99.5% availability, strong consistency for payments

OUTPUT:

SCENARIO 1: Black Friday Traffic Spike
STIMULUS: Traffic increases 10× (100 → 1000 req/sec)
ENVIRONMENT: App Runner at 50% capacity, database at 60% CPU
RESPONSE: System should auto-scale App Runner instances; maintain <500ms p99 latency during scale-up
RESPONSE MEASURE: Latency stays <500ms for first 2 min, drops back to <100ms after scale-up completes (5 min)
EXPECTED RESULT: App Runner auto-scales from 5 → 20 instances within 2 min; database becomes bottleneck after 10 min (needs read replicas or caching tuning)
PASS/FAIL: PARTIAL PASS — Latency goal met for first 2 min, but database is limiting factor beyond 10 min traffic sustained
MITIGATION: Add read replicas for product catalog queries; implement circuit breaker for slow database queries; add caching layer tuning

SCENARIO 2: Database Failure (RDS Multi-AZ Failover)
STIMULUS: Primary database instance fails
ENVIRONMENT: 500 concurrent users, 100 active transactions
RESPONSE: Failover should complete in <30 sec; affected transactions should either commit or rollback cleanly (no partial state)
RESPONSE MEASURE: App Runner should continue accepting requests after 30 sec; error rate should spike <5 sec then drop to <0.1%
EXPECTED RESULT: RDS Multi-AZ automatic failover takes ~30 sec; during failover, connection pool in App Runner experiences 5–10 sec of errors; in-flight transactions may be lost (connection reset)
PASS/FAIL: FAIL — Customers experience errors; lost transactions not automatically retried
MITIGATION: Implement transaction retry logic in App Runner; add connection pool recovery timeout; test failover quarterly

SCENARIO 3: Payment Processing During Partial Outage
STIMULUS: Redis cache fails; database is slow but responding
ENVIRONMENT: 1K concurrent requests, cache offline, database CPU at 80%
RESPONSE: Payment transactions should not use cache (must hit database); should complete in <1 sec p99 for strong consistency
RESPONSE MEASURE: Payment success rate >99.9%; latency <1 sec p99; no payment duplicates
EXPECTED RESULT: App Router routes to uncached payment service; database handles 500 concurrent payment queries at 60ms each; 40ms latency + network = 100ms total (good); no caching violations
PASS/FAIL: PASS — Strong consistency preserved; latency acceptable under degraded conditions

SCENARIO 4: Deployment During Peak Load
STIMULUS: Blue-green deployment happens while traffic is 500 req/sec
ENVIRONMENT: App Runner serving full load, canary deployment starting
RESPONSE: Traffic should reroute to green instance without dropping requests
RESPONSE MEASURE: Error rate during deploy <0.01%; latency p99 stays <200ms
EXPECTED RESULT: App Runner blue-green handles 5–10 sec of routing delays; connection draining prevents in-flight requests from being dropped
PASS/FAIL: PASS — Zero downtime deployment confirmed in staging
```

---

### Example: Weak Output (and How to Remediate)

```
INPUT (Same context):
Architecture: Monolith + Managed Services
NFR: <100ms p99 latency, 99.5% availability, strong consistency for payments

WEAK OUTPUT (Vague measures, generic expectations):

SCENARIO 1: High Traffic
STIMULUS: Traffic spikes
ENVIRONMENT: System under load
RESPONSE: System should handle it well
RESPONSE MEASURE: Performance should be good
EXPECTED RESULT: Likely handles load OK
PASS/FAIL: Unclear
MITIGATION: Maybe add more servers

SCENARIO 2: Database Failure
STIMULUS: Database goes down
ENVIRONMENT: Normal operations
RESPONSE: System should keep working
RESPONSE MEASURE: System stays up
EXPECTED RESULT: We hope failover works
PASS/FAIL: Unknown; haven't tested
MITIGATION: We'll figure it out

SCENARIO 3: Security Issue
STIMULUS: Someone attacks the system
ENVIRONMENT: Normal conditions
RESPONSE: System should be secure
RESPONSE MEASURE: No data loss
EXPECTED RESULT: Our encryption will stop them (probably)
PASS/FAIL: Speculation
MITIGATION: TBD

---

WHY THIS IS WEAK:
1. ❌ Vague stimulus — "Traffic spikes" vs. "10× from 100 → 1000 req/sec"
2. ❌ Unmeasurable response measures — "performance should be good" vs. "<500ms p99"
3. ❌ Generic environments — No actual values (load, concurrency, CPU %)
4. ❌ Speculative expected results — "likely handles," "we hope," "probably"
5. ❌ No concrete PASS/FAIL criteria — "unclear," "unknown," "speculation"
6. ❌ Missing architecture-specific analysis — Doesn't reference App Runner auto-scaling or RDS Multi-AZ
7. ❌ No mitigation specifics — "maybe add servers," "figure it out later"
8. ❌ Security scenario is untestable — No specific threat model or defense mechanism identified

---

REMEDIATION:

**Requirement:** Every scenario must be testable with specific, measurable response criteria.

**Actions:**
1. Stimulus: Quantify the event (traffic: 100 → 1000 req/sec; not "spikes")
2. Environment: Specify system state (App Runner 50% capacity, DB 60% CPU, 500 concurrent users)
3. Response: Name the expected system behavior (auto-scale App Runner; maintain latency target)
4. Response Measure: Use concrete metrics:
   - Latency: "p99 latency <500ms for first 2 min; <100ms after scale-up"
   - Availability: "Error rate <0.01%; no dropped connections"
   - Consistency: "No payment duplicates; transaction rollback clean"
5. Expected Result: Architecture-specific (reference your actual components):
   - App Runner: "Auto-scales from 5 → 20 instances in 2 min (AWS documented behavior)"
   - RDS: "Detects slow queries; connection pool retries; or times out if lag >10 sec"
   - Cache: "Redis eviction follows LRU; no data corruption"
6. PASS/FAIL: Binary decision with thresholds (e.g., "PASS if latency <500ms for sustained 10 min; FAIL if exceeds")
7. Mitigation: Specific implementation (not "TBD"):
   - "Add read replicas: increases cost $2K/month; latency improves to 150ms p99"
   - "Implement circuit breaker: 2-week dev effort; failure rate drops from 2% to 0.01%"
8. Security scenarios: Threat-specific:
   - Instead of "someone attacks": "SQL injection attempt on search endpoint"
   - Expected defense: "Parameterized queries + input validation"
   - PASS/FAIL: "PASS if injection blocked; malicious query rejected before reaching DB"

**Regenerate with:**
- Specific stimulus values (10×, 30 sec, 1K concurrent, etc.)
- Measurable response measures (ms, %, count, minutes)
- Concrete environment values (actual system state, not vague)
- Architecture-specific expected results (name actual components: App Runner, RDS, Redis, etc.)
- Binary PASS/FAIL with numerical thresholds
- Actionable mitigation (cost, effort, specific changes, not "TBD")
- Security scenarios tied to threat models (not generic "be secure")
```

---

### Validation Checklist

- [ ] **Scenarios are testable** (you could write an automated test for each)
- [ ] **Response measures are specific** (milliseconds, percentages, counts—not adjectives)
- [ ] **Expected results are realistic** (not optimistic; acknowledge where architecture might struggle)
- [ ] **Mitigation paths are clear** (if scenario fails, you know what to fix)
- [ ] **Scenarios cover operational realities** (not just ideal conditions)

---

## ✓ Prompt 4: Risk Analysis Per Option

### Use This When
You've shortlisted 2–3 architectures and want to identify hidden risks: team skill gaps, operational complexity, failure modes that only show up at scale, and assumption violations.

### When NOT to Use
Don't use before narrowing options; start with Prompts 1-2 first.

### Prerequisites
- Shortlisted architecture options
- Team size and experience level
- Deployment environment and operational maturity
- Known constraints that could become violations (e.g., "budget is tight; cost overruns are risky")

**Validation Rule:** If architecture description is missing, output will say `[INPUT MISSING: Architecture Description]`. If team context is thin, output will be generic; that's acceptable—use it to drive discussion on which risks matter most. Do NOT proceed silently with made-up assumptions about team skill.

### Input Example

```
ARCHITECTURE: Microservices (API Service, Search Service, Payment Service)
TEAM: 8 engineers (2 frontend, 4 backend, 1 SRE, 1 QA)
OPERATIONAL MATURITY: Intermediate (CI/CD in place, basic monitoring, no chaos testing)
DEPLOYMENT: AWS ECS, manual rollbacks (not automated)
KNOWN CONSTRAINTS: Budget is $150K/year; team has no microservices experience
```

---

### The Prompt

```text
Analyze this architecture for hidden risks. For each risk, provide:

1. RISK NAME (specific, not generic)
2. TRIGGER (what conditions make this risk real?)
3. PROBABILITY (likely, possible, rare—not percentages)
4. IMPACT (what breaks? How many users affected? Revenue impact?)
5. DEPENDENCY (does this risk depend on other risks? E.g., "only happens if monitoring fails AND deployment automation breaks")
6. ROOT CAUSE (why does this architecture create this risk?)
7. EARLY WARNING (what signal should trigger investigation before failure?)
8. MITIGATION (how to reduce probability or impact?)
9. ACCEPTANCE RULE (under what conditions is this risk acceptable?)

Cover:
- Team skill gaps
- Operational complexity (deployment, monitoring, incident response)
- Failure modes at scale
- Cost overruns
- Data consistency issues
- Security assumptions
- Vendor lock-in
- Integration complexity

***
[PASTE ARCHITECTURE AND CONTEXT HERE]
```

---

### Example Output

```
RISK 1: Service Dependency Cascade
RISK: If API Service becomes slow, it causes Search Service to timeout and queue backlog

TRIGGER: Traffic spike (1000 → 10K req/sec) + database optimization not complete

PROBABILITY: Possible (traffic spikes happen; DB optimization not scheduled yet)

IMPACT: Search latency increases to 5+ seconds; users see timeout errors; estimated 10% of searches fail

DEPENDENCY: Happens only if monitoring doesn't alert on API Service latency increase

ROOT CAUSE: Synchronous API call from Search Service to API Service; no circuit breaker; no timeout

EARLY WARNING: API Service latency spike (>200ms) should trigger alert within 30 sec

MITIGATION:
- Add circuit breaker (return cached results if API is slow)
- Add timeout (fail fast instead of queue backlog)
- Add async fallback (queue search request if API is down)
- Load test with 10K req/sec to validate fix

ACCEPTANCE RULE: Acceptable if we implement circuit breaker + monitoring alert before launch

RISK 2: Team Operational Burden
RISK: SRE (1 person) becomes on-call for 3 microservices; incident response takes 30+ min

TRIGGER: Any production incident (database slow, service crash, deployment rollback)

PROBABILITY: Likely (one-person SRE team is stretched thin)

IMPACT: Mean time to resolution (MTTR) is 30–60 min; SRE burnout within 6 months

DEPENDENCY: No automated incident response; no runbooks; alerting is noisy

ROOT CAUSE: Microservices multiply complexity; small SRE team can't handle distributed debugging

EARLY WARNING: SRE response time >15 min for alerts; post-mortems show "couldn't find root cause quickly"

MITIGATION:
- Hire 2nd SRE before launch OR reduce microservices to 2 services
- Implement automated recovery (auto-rollback on deployment failure, circuit breaker, connection pooling)
- Write runbooks for top 5 incident scenarios
- Set up distributed tracing (X-Ray or Jaeger) to accelerate debugging

ACCEPTANCE RULE: Acceptable if SRE tools + runbooks reduce MTTR to <10 min for 95% of incidents
```

---

### Example: Weak Output (and How to Remediate)

```
INPUT (Same context):
Architecture: Microservices (API Service, Search Service, Payment Service)
Team: 8 engineers (2 frontend, 4 backend, 1 SRE, 1 QA)
Operational maturity: Intermediate; team has no microservices experience

WEAK OUTPUT (Generic risks, no specifics):

RISK 1: Complexity
RISK: Microservices are complex
TRIGGER: When complexity is high
PROBABILITY: High
IMPACT: System might break
EARLY WARNING: Things might fail
MITIGATION: Try to keep it simple

RISK 2: Team Issues
RISK: Team might not know microservices
TRIGGER: Team lacks experience
PROBABILITY: Likely
IMPACT: We'll have to learn
MITIGATION: Read some docs

RISK 3: Cost Overruns
RISK: Costs might be higher than expected
TRIGGER: More services than planned
PROBABILITY: Unknown
IMPACT: Budget might exceed $150K
MITIGATION: Monitor costs

---

WHY THIS IS WEAK:
1. ❌ Generic risk names — "Complexity" applies to everything; should name specific risk
2. ❌ Triggers are vague — "when complexity is high" is circular; triggers should be concrete conditions
3. ❌ No probability justification — "Likely" for team learning curve but no reasoning
4. ❌ Impact too vague — "System might break" has no quantification (how many users? revenue loss?)
5. ❌ No dependency analysis — Don't explain when risks compound (e.g., complexity + team inexperience)
6. ❌ No root cause — Don't explain why this architecture creates the risk
7. ❌ No early warning metrics — "Things might fail" is not detectable until it's too late
8. ❌ Mitigation too generic — "Try to keep it simple," "Read some docs" are not actionable
9. ❌ Acceptance rules missing — No criteria for when team should accept this risk
10. ❌ No cost quantification — Budget overrun not tied to specific causes (how many extra services?)

---

REMEDIATION:

**Requirement:** Risks must be specific, quantifiable, and tied to architecture + team context.

**Actions:**
1. Risk name: Specific failure mode (not "Complexity")
   - ✓ Service Dependency Cascade (specific chain of failures)
   - ✗ Complexity (too generic)
2. Trigger: Concrete conditions that activate the risk
   - ✓ "Traffic spike >5K req/sec + Search Service queries API Service synchronously + no circuit breaker"
   - ✗ "When complexity is high" (vague)
3. Probability: Justified with evidence
   - ✓ "Likely (observed in 70% of microservices projects; this team has no experience)"
   - ✗ "Likely" (no reasoning)
4. Impact: Quantified with business consequence
   - ✓ "10% of search requests fail; ~$2K revenue loss per incident; SRE spends 4 hours debugging"
   - ✗ "System might break" (unmeasurable)
5. Dependency: Explain when risks compound
   - ✓ "Only happens if: (1) traffic spikes + (2) API Service slow + (3) no circuit breaker + (4) monitoring misses alert"
   - ✗ "Just happens" (isolated)
6. Root cause: Why this architecture creates the risk
   - ✓ "Microservices introduce synchronous inter-service calls; without circuit breakers, one slow service cascades to others"
   - ✗ "Microservices are complex" (not root cause)
7. Early warning: Detectable signal before failure
   - ✓ "API Service latency spike >200ms (monitored every 10 sec); should alert within 30 sec"
   - ✗ "Things might fail" (undetectable)
8. Mitigation: Specific, actionable, with cost/effort
   - ✓ "Add circuit breaker to Search→API call (2-day dev effort, $0 cost); reduces cascade risk from 70% to <5%"
   - ✗ "Try to keep it simple" (not actionable)
9. Acceptance rule: Explicit criteria for accepting risk
   - ✓ "Acceptable if circuit breaker + monitoring alert both implemented before launch"
   - ✗ "Monitor and see" (no criteria)
10. Cost tie-in: Link to budget constraint
    - ✓ "If we add 2 new services vs. planned 3, cost increases $10K/year (monitoring + orchestration); cumulative risk budget: $20K"
    - ✗ "Costs might be higher" (unquantified)

**Regenerate with:**
- Specific risk names (e.g., "Service Dependency Cascade," "SRE Operational Burden," "Cost Overrun from Monitoring")
- Concrete triggers (not vague conditions)
- Probability justified with evidence or precedent
- Impact quantified (revenue loss, MTTR, user count affected, SRE hours)
- Dependency analysis showing when risks compound
- Root cause tied to architecture + team constraints
- Early warning signals that are detectable/measurable
- Mitigation with specific effort + cost
- Acceptance rules tied to launch criteria
- Cost overruns quantified and tied to budget constraint
```

---

## ✓ Prompt 5: Architecture vs. Constraint Check

### Use This When
You want a rapid elimination filter: which architectures violate which constraints? Build a quick compliance matrix without deep analysis.

### When NOT to Use
Skip if you've already validated compliance in earlier prompts.

### Prerequisites
- Multiple architecture options
- Explicit constraints (not vague; with numerical targets)

---

### Input Example

```
ARCHITECTURES:
1. Monolith
2. Microservices
3. Event-driven
4. Serverless

CONSTRAINTS:
- Team size: 4 people max
- Data residency: EU only
- Latency: <100ms p99 for core APIs
- Deployment: On-premises only (no AWS)
```

---

### The Prompt

```text
Create a compliance matrix. Rows: architectures. Columns: constraints.

For each cell: FITS / VIOLATES / REQUIRES VALIDATION

If VIOLATES or REQUIRES VALIDATION, add ONE SENTENCE explaining why.

Do not estimate or guess. If constraint compliance is unclear, mark REQUIRES VALIDATION and explain what information is needed.

***
[PASTE ARCHITECTURES AND CONSTRAINTS HERE]
```

---

### Example Output

```
| Architecture | Team Size (4) | EU-Only Residency | Latency <100ms p99 | On-Premises Deploy | Verdict |
|---|---|---|---|---|---|
| **Monolith** | FITS | FITS (single region) | FITS (local DB) | REQUIRES VALIDATION (depends on on-prem infrastructure setup) | 3/4 constraints clear; 1 needs infrastructure assessment |
| **Microservices** | VIOLATES (needs 8+ engineers for 3+ services) | FITS | REQUIRES VALIDATION (inter-service latency adds hops) | VIOLATES (complex K8s on-prem setup, not standard) | Violates 2 constraints; high risk |
| **Event-Driven** | FITS (async reduces coordination) | FITS | VIOLATES (eventual consistency adds latency, conflicts with <100ms requirement) | FITS | Violates 1 constraint (latency) |
| **Serverless** | FITS | FITS (AWS region within EU) | FITS (managed, optimized) | VIOLATES (serverless requires cloud, not on-prem) | Violates 1 constraint (on-prem); requires cloud exception |
```

---

### Example: Weak Output (and How to Remediate)

```
WEAK OUTPUT (Missing reasoning, unsure compliance):

| Architecture | Team Size (4) | EU-Only Residency | Latency <100ms p99 | On-Premises Deploy | Verdict |
|---|---|---|---|---|---|
| **Monolith** | Maybe | Probably | Likely | Unclear | Uncertain |
| **Microservices** | No | Yes | No | No | Bad fit |
| **Event-Driven** | Maybe | Yes | No | Yes | Uncertain |
| **Serverless** | Yes | No | Yes | No | Doesn't fit |

Comment: "All options have tradeoffs. Need more discussion."

---

WHY THIS IS WEAK:
1. ❌ Vague compliance: "Maybe," "Probably," "Unclear" — should be FITS/VIOLATES/REQUIRES VALIDATION
2. ❌ No reasoning for any cell — can't explain why monolith is "unclear" for on-prem
3. ❌ Contradictions unchecked — Serverless says "No" for EU residency but "Yes" for latency (contradictory)
4. ❌ No investigation of "Unclear" — If compliance unclear, state what info is missing
5. ❌ Verdict doesn't match cells — Says "Doesn't fit" for serverless, but has 2 YESes
6. ❌ No elimination criteria — Output doesn't tell you which architectures are viable

---

REMEDIATION:

**Requirement:** Compliance must be binary (FITS/VIOLATES/REQUIRES VALIDATION) with reasoning.

**Actions:**
1. Replace all vague terms:
   - "Maybe" → FITS or VIOLATES or REQUIRES VALIDATION (pick one, explain why)
   - "Unclear" → REQUIRES VALIDATION (state what info is needed)
   - "Probably" → FITS (with reasoning) or REQUIRES VALIDATION
2. For each cell, add one sentence explaining why:
   - FITS: "Monolith on-prem fits because: standard deployment model, no cloud dependency"
   - VIOLATES: "Serverless on-prem violates because: serverless fundamentally requires cloud provider"
   - REQUIRES VALIDATION: "Microservices on-prem needs validation because: depends on on-prem K8s infrastructure quality"
3. Verify consistency: If Event-Driven says "No" for latency, explain why (eventual consistency > 100ms)
4. Elimination logic: Based on VIOLATES cells, eliminate architectures with >1 violation
5. Count: Summarize FITS, VIOLATES, REQUIRES VALIDATION per architecture

**Regenerate with:**
- Binary compliance (FITS / VIOLATES / REQUIRES VALIDATION only)
- One-sentence reasoning for every cell
- No contradictions or vague entries
- Clear elimination: which architectures have fatal violations?
- Which require investigation before proceeding?
```

---

## ✓ Prompt 6: Architectural Pattern Justification

### Use This When
You need to justify to stakeholders why you chose Microservices over Monolith (or vice versa). You want a clear narrative that ties the choice back to constraints, not just "microservices are modern."

### When NOT to Use
Don't use before deciding which pattern to use (decision must come first).

### Prerequisites
- Chosen architecture
- Rejected alternatives
- Constraints that drove the decision
- Stakeholder concerns (cost, speed, team velocity, scalability)

**Validation Rule:** If constraints are missing, output will say `[INPUT MISSING: Constraints]`. Provide specific values (not vague: "$150K budget", "6-month timeline", "8-person team"). If stakeholder concerns aren't listed, generic objections will appear; that's acceptable—use it to drive discussions with decision-makers.

### Input Example

```
CHOSEN: Monolith + Managed Services
REJECTED: Microservices (3 services), Event-driven

DECISION DRIVERS:
- Team: 8 engineers (not specialized)
- Timeline: 6 months to launch
- Budget: $150K/year
- Expected scale: 10K → 100K users over 18 months

STAKEHOLDER CONCERNS:
- CTO: "Monoliths don't scale"
- CFO: "Budget is tight; want lowest cost"
- PM: "Need to ship fast"
- SRE (1 person): "Don't want operational complexity"
```

---

### The Prompt

```text
Write a 2-page justification document explaining why the chosen architecture is the best fit 
for THIS project (not in general).

Structure:
1. Decision summary (1 paragraph)
2. Constraint mapping (table: constraint vs. why chosen architecture is best fit)
3. Stakeholder concern response (address each concern from the input)
4. Rejected alternatives + why (for each rejected option, explain the tradeoff)
5. Success criteria (how will we know this choice was right? What metrics matter?)
6. Upgrade path (if assumptions change, when/how do we reconsider this choice?)

CRITICAL RULE: Every statement must reference a constraint or stakeholder concern. 
Do NOT use generic reasoning ("monoliths are simple"). Use project-specific reasoning 
("with 6 months and 8 engineers, monolith lets us ship by month 4 and still have buffer").

***
[PASTE CHOSEN ARCHITECTURE, REJECTED ALTERNATIVES, CONSTRAINTS, AND STAKEHOLDER CONCERNS HERE]
```

---

### Example Output Section

```
DECISION SUMMARY:
We chose Monolith + Managed Services because it aligns with our strongest constraints: 
6-month timeline, 8-person team, $150K budget. This choice enables the team to ship 
features weekly, keeps infrastructure costs at $40K/year (vs. $65K for microservices), 
and lets our single SRE operate the system without 24/7 incident response burden.

CONSTRAINT MAPPING:
| Constraint | Value | Why Monolith Wins |
|---|---|---|
| Timeline | 6 months | Single codebase = faster feature integration; no service coordination overhead |
| Team size | 8 | All engineers work on single codebase; microservices would require service owners, complicating handoffs |
| Budget | $150K/year | Monolith infrastructure ~$40K/year; microservices ~$65K/year |
| SRE capacity | 1 FTE | Monolith has fewer deployment points; microservices would require 24/7 incident response from 1 person (burnout risk) |
| Scale | 10K → 100K users | Monolith handles 50K users without changes; can add read replicas at 75K; if higher, migrate to microservices in month 12 |

CTO CONCERN: "Monoliths don't scale"
RESPONSE: True in general. Specific to our project: we'll hit scaling limits around 50K concurrent users. Our growth forecast is 10K → 100K over 18 months. Monolith gives us 12 months of runway; at month 12 (or when we hit scaling issues), we have proven product-market fit and can invest in microservices migration. This is a deliberate choice, not a blind spot.

...
```

---

### Example: Weak Output (and How to Remediate)

```
WEAK OUTPUT (Generic justification, no constraint grounding):

DECISION SUMMARY:
We chose monolith because monoliths are simple and easy to deploy. Microservices are too complex for this stage. We don't need the scalability of microservices right now.

CONSTRAINT MAPPING:
| Constraint | Why Monolith Wins |
|---|---|
| Timeline | Monoliths are fast to build |
| Team | We're familiar with monoliths |
| Budget | Monoliths are cheap |
| Scale | We'll scale later if needed |

STAKEHOLDER CONCERNS:
CTO: "Monoliths don't scale" → We can always refactor later
CFO: "Keep costs low" → Monoliths are cheaper
PM: "Ship fast" → Monoliths are simple
SRE: "Operational complexity" → Monoliths are straightforward

UPGRADE PATH:
If we need to scale, we can migrate to microservices eventually.

---

WHY THIS IS WEAK:
1. ❌ No constraint-specific reasoning — "Monoliths are simple" is generic, not tied to YOUR 6-month timeline
2. ❌ Stakeholder responses are dismissive — "We can refactor later" doesn't address CTO's real concern (scalability risk)
3. ❌ Upgrade path vague — "Eventually" and "if we need" have no trigger (when? at what user count?)
4. ❌ No rejected alternatives analysis — Why is monolith better than event-driven? No comparison
5. ❌ No success criteria — How will you know this choice was right?
6. ❌ Constraint mapping lacks specifics — "Budget is cheaper" by how much? $20K? $50K difference?
7. ❌ No data to back claims — "Monoliths are fast" unsupported by timeline or team composition
8. ❌ Scale handling glossed over — "Scale later" without mentioning 18-month growth forecast or scaling limits

---

REMEDIATION:

**Requirement:** Every statement must reference a specific constraint or stakeholder concern with quantified reasoning.

**Actions:**
1. Decision summary: Tie to numbered constraints
   - ✓ "Given 6-month timeline + 8-person team + $150K budget, monolith enables weekly features (Constraint 1), keeps infrastructure $40K/year (Constraint 3), and lets 1 SRE operate without cascading failures (Constraint 4)"
   - ✗ "Monoliths are simple" (generic)
2. Constraint mapping: Add specific reasoning
   - ✓ "Timeline 6 months: Single codebase = no service coordination meetings; can parallelize feature work by module, not by service"
   - ✗ "Monoliths are fast to build" (unsupported)
3. Stakeholder concern response: Address the underlying worry
   - ✓ CTO "Monoliths don't scale" → "True. Our growth forecast: 10K → 100K users over 18 months. Monolith scales to 50K users without changes (month 12 at projected growth). At that point, we have PMF, can invest in migration. This is deliberate staging, not blindness."
   - ✗ "We can refactor later" (dismissive, no data)
4. Rejected alternatives: Compare on constraints
   - ✓ Microservices: "Requires 8+ engineers per service team (we have 8 total); would need 4 months onboarding for inter-service testing patterns"
   - ✗ Just say "too complex"
5. Success criteria: Measurable thresholds
   - ✓ "Success: Ship features weekly (target: 1–2 per week); ops burden <0.5 FTE (SRE <30 hrs/week); cost <$50K/year"
   - ✗ "It works well"
6. Upgrade path: Explicit triggers
   - ✓ "Upgrade to microservices if: (1) we hit 50K concurrent users before month 12, OR (2) feature development parallelism blocked by monolith coupling"
   - ✗ "Eventually if needed"
7. Quantified tradeoff: Show the numbers
   - ✓ "Monolith: $40K/year infrastructure + 1 SRE = $150K total. Microservices: $65K infrastructure + 2 SRE = $200K total. Difference: $50K/year for 18 months = $75K opportunity cost if monolith adequate."
   - ✗ "Monolith is cheaper"

**Regenerate with:**
- Decision summary tied to specific constraints (timeline, budget, team, scale)
- Constraint mapping with quantified reasoning (not generic claims)
- Stakeholder concern responses that address underlying risks
- Rejected alternatives with tradeoff analysis
- Success criteria with measurable targets
- Upgrade path with explicit triggers (not "someday")
- Cost tradeoff with numbers (not vague "cheaper")
```

---

## How to Use

### Initial Exploration

1. Run **Constraint-Driven Architecture Generation** with your requirements + constraints
   - Input: Clear requirements + constraints inventory from Section 1
   - Output: 3–5 viable architecture options that fit *your* constraints
   
2. For each option, run **Quality Attribute Scenarios** to stress-test against your NFRs
   - Input: One architecture option + your quality attribute targets
   - Output: Scenarios showing how the option performs under load, failure, scale-up, etc.

3. Run **Tradeoff Analysis** — constraint-grounded comparison
   - Input: All options + your constraints + quality attributes
   - Output: Tradeoff table with *specific* rationale (not generic 1–5 ratings)

### Constraint Filtering

4. For each constraint, ask: "Which architectures violate this constraint?"
   - Use prompt: **Architecture vs. Constraint Check**
   - Output: Elimination list (e.g., "Microservices violates 3-person team size constraint")
   - **Decision:** If only one option remains, stop exploring. If multiple remain, compare tradeoffs.

5. Run **Risk Analysis** on remaining options
   - Input: Top 2–3 viable options + your operational maturity
   - Output: Failure modes, team skill gaps, hidden dependencies per option

---

## Customization Guidance

### By Team Size

- **Solo architect or 2-3 person team:** Filter *toward* monolith, event-driven; filter *against* microservices (operational complexity too high)
- **5-10 person team:** Monolith + selective microservices; avoid complex event-driven systems
- **20+ person team:** All options viable; optimize by organizational structure and deployment constraints

### By Operational Maturity

- **Startup (no SRE, no observability):** Favor serverless, managed services, monolith
- **Mid-scale (SRE team emerging, basic monitoring):** Monolith + event-driven; single microservice for scaling bottleneck
- **Enterprise (mature SRE, canary deploys, observability):** All options viable; optimize by cost and organization

### By Regulatory Domain

- **Finance/Healthcare:** Constraint: Data residency + audit trails. Filter options that violate data locality.
- **High-frequency trading:** Constraint: Sub-millisecond latency. Filter options that add network hops.
- **Government:** Constraint: Specific infrastructure (on-prem, air-gapped). Filter cloud-only options.

### By Time-to-Market Pressure

- **Ship in 3 months:** Favor architectures that match your team's experience (reduce learning curve)
- **Ship in 6+ months:** Consider architectures that build internal capability (e.g., event-driven if strategic)

---

## Quality Checks

### Before Using Architecture Output

- [ ] Every option has explicit constraint violations or fits (not vague)
- [ ] Tradeoff ratings are justified by *specific constraint impact*, not generic scoring (e.g., "Microservices cost +30% due to orchestration + monitoring" not "cost = 3/5")
- [ ] Every architecture has named risks (e.g., "requires 3+ SRE headcount" or "assumes async messaging expertise")
- [ ] No invented technologies or capabilities appear (e.g., "we'll use AI to auto-scale" without detail)

### Red Flags (Hallucination Indicators)

- ❌ Generic architectures appear for every problem ("here are 4 standard options") — should be constraint-driven, not template-based
- ❌ Tradeoff ratings lack reasoning ("Scalability = 4/5" without saying why) — mark as `[REQUIRES GROUNDING]`
- ❌ Quality Attribute Scenarios use made-up measures ("Response time = 'good'" instead of milliseconds) — mark as `[SPECULATIVE]`
- ❌ Team skill requirements are assumed, not stated ("assumes DevOps team" when you said solo architect) — flag as `[CONSTRAINT VIOLATION]`

### Success Indicators

- ✅ Only architectures that respect stated constraints appear
- ✅ Tradeoff analysis cites specific constraint impact (cost, latency, complexity, skill requirement)
- ✅ Each architecture includes explicit *risk* section (failure modes, team skill gaps, operational dependencies)
- ✅ Quality Attribute Scenarios reference your actual NFR targets (not generic benchmarks)

---

## Next Steps

After exploration:
1. Select top 2–3 architectures for deeper analysis
2. Move to **03 — Adversarial Review** with shortlist
3. Use adversarial review to pressure-test assumptions and catch blind spots
4. Document all tradeoff decisions as ADRs in Section 4

---

## Resources

- Shared constraints vocabulary: `/shared/constraints-vocabulary.md`
- ISO 25010 quality attributes: `/shared/quality-attributes.md`
- Constraint-grounding rubric: `/shared/constraint-grounding-rubric.md` (use to validate tradeoff reasoning)
- Anti-slop checklist: `/shared/anti-slop-checklist.md`

---

**Status:** Complete (6 prompt cards, constraint-grounded scoring, validation rules, guardrails applied, weak examples with remediation added to all prompts)
**Last Updated:** 2026-06-02 (Neo — Weak Examples + Squad Files Added)

