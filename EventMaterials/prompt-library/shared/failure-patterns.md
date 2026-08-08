# Failure Patterns Reference

Common failure modes and hallucination patterns to watch for in AI-generated architecture content.

---

## Purpose

This reference documents failure patterns so you can recognize and catch them early. Use it as a pattern-matching guide during code review and architecture validation.

---

## Category 1: Constraint-Ignoring Failures

### Pattern: "Generic Best Practice" Recommendation
**What it looks like:**
```
"Use microservices because they scale better and are easier to maintain."
```

**Why it fails:**
- References no constraints from your project
- Could apply to any project
- Hides tradeoffs (learning curve, operational complexity)
- Generic claim unsupported by evidence

**Detection:** Recommendation doesn't reference team size, timeline, budget, or scale constraints

**Fix:**
```
"Use microservices because you have 5 independent teams (enables parallel deployments without waiting for coordinated releases; satisfies feature velocity requirement of 2–3 features/sprint). Monolith would require all teams to deploy together (reduces velocity to ~1 feature/sprint)."
```

---

### Pattern: Technology Choice Without Constraint Grounding
**What it looks like:**
```
"Choose Node.js for the API server because it's modern and has a large ecosystem."
```

**Why it fails:**
- "Modern" is marketing buzzword, not a constraint
- "Large ecosystem" doesn't mean it fits YOUR constraints
- No reference to team skill, timeline, or performance requirements

**Detection:** No mention of team expertise, learning curve, or specific requirement it solves

**Fix:**
```
"Choose Node.js because: (1) 4 of 8 engineers have 5+ years Node.js experience (TTM constraint: no learning curve). (2) Non-blocking I/O naturally handles concurrent requests (latency requirement: <500ms p99). (3) Alternative: Go requires 3-month team learning (violates 6-month TTM)."
```

---

### Pattern: Missing Constraint Conflict
**What it looks like:**
```
Architecture proposed: Microservices with Kubernetes
Constraints: 6-month TTM, 2-person team, $50K annual budget
Grounding: [Completely absent; no mention of constraints]
```

**Why it fails:**
- Kubernetes learning: 3–6 months (violates TTM)
- Operational burden: 1 FTE SRE minimum (no budget for it; 2-person team can't spare capacity)
- Cost: Self-managed K8s ~$500K year (violates $50K budget)
- Red flags not mentioned; conflict hidden

**Detection:** Recommendation doesn't explicitly check constraints; conflicts not addressed

**Fix:**
```
"Kubernetes rejected because: (1) Learning curve 3–6 months (violates 6-month TTM; team would be learning, not shipping). (2) Operational burden requires 1 FTE SRE (no budget). (3) Cost $500K+ annually (violates $50K budget). Recommendation: AWS App Runner (managed Kubernetes) fits all constraints: no learning required, $2K/month, no ops expertise needed."
```

---

## Category 2: Invented Capabilities Failures

### Pattern: Automatic Failover Without Mechanism
**What it looks like:**
```
"The system automatically recovers from failures, ensuring high availability."
```

**Why it fails:**
- "Automatic" is claimed but not specified
- What detects failure? What action does it take?
- Could mean automatic restart (not true HA) or automatic failover (requires replication)
- Confidence unsupported

**Detection:** Search for "automatic," "seamless," "self-healing"; ask "specifically, how?"

**Fix:**
```
"RDS Multi-AZ provides automatic failover: (1) Primary database fails → Route 53 health check detects (within 30 seconds) → DNS redirects to standby replica. (2) Tested: Simulated primary failure in staging; failover completed in 35 seconds. (3) RTO (Recovery Time Objective): ~1 minute; acceptable for 99.5% SLA."
```

---

### Pattern: "Zero-Downtime Deployment" Without Details
**What it looks like:**
```
"We'll use zero-downtime deployment to ensure customers never experience outages."
```

**Why it fails:**
- "Zero-downtime" is goal, not mechanism
- Blue-green? Rolling restart? Feature flags? Each has different tradeoffs
- Claim of "zero" downtime is overconfident; always some risk
- No mention of tested procedure or rollback

**Detection:** Claims of "zero" or "seamless" or "guaranteed" without how-to

**Fix:**
```
"Deployment strategy: Blue-green deployment (5-minute cutover window). Procedure: (1) Deploy new code to standby environment (10 min). (2) Run smoke tests (2 min). (3) Flip load balancer to standby (1 min, requires DNS change). (4) If failure, flip back (1 min). Tested: 3 times in staging; cutover successful. Risk: DNS propagation can take up to 5 minutes; brief period where some traffic hits old code; mitigated by monitoring + instant rollback if error rate spikes."
```

---

### Pattern: "Self-Optimizing" System Without Specifics
**What it looks like:**
```
"The system self-optimizes by learning from production traffic patterns."
```

**Why it fails:**
- "Self-optimizes" is vague
- No specific optimization algorithm named
- No evidence that it works in practice
- Over-confident claim about "learning"

**Detection:** ML/AI buzzwords without concrete mechanism or evidence

**Fix:**
```
"Database query optimization: Use pg_stat_statements view to identify slow queries. Automated job runs weekly: (1) Identifies queries with >100ms average latency. (2) Suggests missing indexes or query rewrites. (3) Escalates top 3 to SRE for manual review + implementation. Does NOT auto-execute changes (prevents unexpected performance regressions). Tested: Identified 5 missing indexes in 3 weeks; indexes reduced average latency by 40%."
```

---

## Category 3: Ungrounded Confidence Failures

### Pattern: Certainty Without Evidence
**What it looks like:**
```
"This architecture guarantees 99.99% uptime with our constraints."
"This solution eliminates all scaling concerns."
"This approach is proven and industry-standard."
```

**Why it fails:**
- "Guarantees" unsupported; nothing is certain
- "Proven" and "industry-standard" are claims, not evidence
- "Eliminates" is overconfident; every solution has tradeoffs
- No mention of assumptions or edge cases

**Detection:** Superlatives or certainty without caveats

**Fix:**
```
"Availability: RDS Multi-AZ supports 99.95% SLA (measured by AWS). Our 99.5% target exceeds RDS capability by 2x margin. Caveat: This assumes (1) health checks work correctly, (2) failover scripts complete in <1 minute, (3) no cascading failures in application layer. If any assumption fails, confidence drops to 99% or lower. Mitigation: Test failover quarterly; monitor health check latency; implement circuit breakers in app."
```

---

### Pattern: Missing Confidence Qualification
**What it looks like:**
```
"The team can learn Kubernetes and operationalize it in 3 months."
```

**Why it fails:**
- Stated as fact, not probability
- No mention of confidence level (1–5)
- No acknowledgment of uncertainty or risk
- No validation plan if estimate is wrong

**Detection:** Claims stated as fact when they're actually estimates/assumptions

**Fix:**
```
"ASSUMPTION: Team can operationalize Kubernetes in 3 months. Confidence: 2/5 (uncertain).

Evidence FOR: (1) One engineer read Kubernetes book. (2) Similar teams achieved proficiency in 4 months.
Evidence AGAINST: (1) No one has production Kubernetes experience. (2) Learning curve is steep; one engineer might take 6 months. (3) Concurrent project timeline is tight; learning takes capacity from development.

Validation Plan:
- Month 1, week 2: Kubernetes workshop + PoC deployment. Success = PoC in dev cluster with 2 engineers.
- If PoC delayed >1 week: Escalate; consider fallback (AWS App Runner).

Fallback: If Kubernetes adoption stalls, use App Runner (managed Kubernetes) instead; no learning required."
```

---

## Category 4: Repeated Content / Hallucination Failures

### Pattern: Same Assumption Repeated Multiple Times
**What it looks like:**
```
Document line 5: "Team has PostgreSQL expertise."
Document line 45: "Team is experienced with PostgreSQL."
Document line 120: "PostgreSQL expertise available on team."
```

**Why it fails:**
- Same fact stated 3 times in different wording
- Wastes reader attention
- Makes document seem longer/more authoritative than it is
- Hides the fact that only 1 unique piece of information exists

**Detection:** Use find/replace; search for key terms; look for paraphrased repetition

**Fix:**
```
Reference once at top: "Team: 3 backend engineers with 5+ years PostgreSQL expertise each."
Later references: "Team PostgreSQL expertise (documented above)"
Or use assumption ID: "ASSUME-001: Team has PostgreSQL expertise"
```

---

### Pattern: Redundant Risk/Mitigation Documentation
**What it looks like:**
```
Risk Register: "Database might become bottleneck"
Architecture Spec: "Database could be bottleneck if scale 10x"
ADR Consequences: "Database scaling might be an issue"
Assumption Register: "Database can scale to X scale"
```

**Why it fails:**
- Same risk documented 4 different ways
- Reader isn't sure if these are different risks or same risk
- Contradictions might exist (Risk says "mitigated"; Assumption says "unvalidated")
- Document becomes incoherent

**Detection:** Search for keywords across all documents; consolidate

**Fix:**
```
Risk Register (source of truth):
- RISK-001: Database write bottleneck at 100K concurrent users
- Probability: Medium (at 10K+ concurrent, indexing required)
- Mitigation: Sharding by customer_id; planned for year 2

Assumptions Register references Risk-001:
- ASSUME-005: Database can scale to 100K with sharding
- Depends on: RISK-001 mitigation being implemented

ADR references Risk-001:
- "Monolith acceptable because database bottleneck is known and documented in RISK-001; mitigation in place"
```

---

## Category 5: Out-of-Scope Details in Wrong Document

### Pattern: Implementation Details in Architecture Decision Record
**What it looks like:**
```
ADR — Architecture Decision Record
Title: Use PostgreSQL for the database

Decision: [Correct; architectural level]
...
Detail: "Use pg_stat_statements view to monitor query performance.
Create an index on customer_id, created_at columns.
Set max_connections = 500 and shared_buffers = 256MB.
Use connection pooling via PgBouncer."
```

**Why it fails:**
- Implementation tuning (index names, pool size) belongs in runbooks, not ADRs
- ADR should focus on DECISION level, not tactical level
- Makes ADR cluttered; hides architectural reasoning
- Implementation details change frequently; don't belong in decision record

**Detection:** Search for specifics (config values, library names, SQL statements)

**Fix:**
```
ADR — Keep architectural:
"Decision: Use PostgreSQL for strong consistency requirement. Rationale: [architectural reason]
Consequences: Requires database tuning expertise on team (address in training plan); monitoring infrastructure needed (address in ops spec)"

Runbook — Implementation details:
"PostgreSQL Tuning:
1. Monitor: pg_stat_statements shows slow queries
2. Indexing strategy: See [separate indexing guide]
3. Configuration: max_connections = 500 (document reasoning)
4. Connection pooling: PgBouncer for horizontal scaling"
```

---

### Pattern: Business Process in Process Flow Document
**What it looks like:**
```
Process Flow: Order Fulfillment
Step 1: Customer places order [CORRECT]
Step 2: Order inserted into database [CORRECT]
Step 3: Inventory manager verifies stock [OUT OF SCOPE]
Step 4: Manager sends approval email [OUT OF SCOPE]
Step 5: Fulfillment team picks items [CORRECT]
```

**Why it fails:**
- Steps 3–4 are business process, not operational flow
- Process flow should describe how SYSTEM operates, not how people coordinate
- Mixes concerns; makes flow confusing
- Hides the actual technical flow

**Detection:** Look for human manual decision points that belong in business docs

**Fix:**
```
Process Flow (System Operation):
Step 1: POST /orders received
Step 2: Order Service validates + reserves inventory (step-by-step system actions)
Step 3: Payment Service authorizes payment
Step 4: Order persisted; OrderCreated event published
Step 5: Fulfillment Service consumes event

Business Process (separate doc):
Step A: Inventory manager reviews low-stock items (business process, not system)
Step B: Manager sends approval for reorder (business, not system)
Step C: Procurement initiates supplier purchase (business, not system)
```

---

## Category 6: Missing Evidence / Validation Failures

### Pattern: Claim Without Citation
**What it looks like:**
```
"PostgreSQL can easily handle 100K concurrent connections."
```

**Why it fails:**
- "Easily" is unqualified
- No evidence cited (test, case study, documentation)
- Different context (PostgreSQL expert's laptop vs. production with your schema)
- Confidence undefined

**Detection:** Look for claims without [citation] or [assumption] flag

**Fix:**
```
"CLAIM: PostgreSQL single instance can handle 100K concurrent connections
EVIDENCE: (1) Tested in our staging environment: Created 1M rows (realistic schema); simulated 100K concurrent connections; achieved <200ms p99 latency. (2) PostgreSQL documentation: "max_connections defaults to 100; can scale to 10K+; practical limit depends on schema, queries, hardware."
CONFIDENCE: 4/5 (tested in our context, but scaling beyond 100K untested)
VALIDATION: Load test in month 1 with production-like data; if performance degrades, escalate for sharding strategy"
```

---

## Anti-Patterns Quick Reference

| Failure | Detection | Fix |
|---------|-----------|-----|
| Generic "best practice" | No constraint reference | Tie every recommendation to specific constraint |
| Invented capability | Search for "automatic," "seamless," "zero-" | Specify exact mechanism; what detects + what action? |
| Ungrounded confidence | Superlatives without caveats | Add assumptions + confidence level (1–5) + validation plan |
| Repeated content | Find/replace duplicate terms | State once + reference by ID elsewhere |
| Out-of-scope detail | Implementation detail in architecture doc | Move to implementation docs; keep arch at decision level |
| Missing evidence | Claim without citation | Cite evidence or flag as [ASSUMPTION] with confidence |
| Conflicting constraints | Multiple contradictory requirements | Prioritize + document why one takes precedence |
| No fallback | "This is the plan" | Always document "if this assumption fails, plan B is..." |

---

## Failure Pattern Checklist

**Before finalizing any architecture document, scan for these patterns:**

- [ ] No generic recommendations ("better," "scalable," "modern" without metrics)
- [ ] No invented capabilities ("automatic" without mechanism)
- [ ] No unqualified confidence ("guarantees," "proven" without evidence)
- [ ] No repeated content (find/replace confirms each fact stated once)
- [ ] No out-of-scope details (implementation specifics in wrong document)
- [ ] No missing evidence (every claim cited or flagged as [ASSUMPTION])
- [ ] No conflicting constraints (priority documented if conflicts exist)
- [ ] Every assumption has fallback plan if proven wrong

**If any check fails: Fix before sharing with stakeholders.**
