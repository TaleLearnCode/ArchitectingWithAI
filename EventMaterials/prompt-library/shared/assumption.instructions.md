# Assumption Register Instructions

Template for documenting assumptions: what we're assuming to be true, why it matters, validation plan, fallback, and ownership.

---

## Purpose

An Assumption Register captures beliefs that affect architecture decisions. By explicitly documenting assumptions, teams can validate them early, catch surprises, and plan fallbacks before the assumption proves wrong.

---

## Why Track Assumptions?

Assumptions are architecture **grenades**: if they explode, architecture falls apart. Examples:

- "Our team can learn Kubernetes in 3 months" — If false, we miss launch window
- "Payment Gateway has 99.99% SLA" — If false (or if network to it is flaky), checkout fails frequently
- "Database will handle 10K writes/sec" — If false, system saturates under peak load
- "Customers will accept eventual consistency" — If false, data freshness concerns derail launch

Assumptions break. Documented assumptions fail gracefully.

---

## Template Structure

### Header
- **Assumption ID:** Unique identifier (e.g., ASSUME-001)
- **Assumption Statement:** One clear, testable claim
- **Category:** Skill / Technical Capability / Vendor / Market / Operational
- **Owner:** Who validates this? (name or role)
- **Confidence Level:** 1–5 (1 = very uncertain; 5 = near certain)
- **Status:** Unvalidated / Validating / Validated / Invalid

### 1. Assumption Statement

**One clear, testable claim.**

**Examples:**

✓ **Good:** "Our 8-person team can learn and operationalize Kubernetes in 3 months without hiring specialists"  
❌ **Weak:** "We can use Kubernetes"

✓ **Good:** "Payment Gateway latency is <200ms p99 under normal load (not during their outages)"  
❌ **Weak:** "Payment Gateway is fast"

✓ **Good:** "PostgreSQL single-node write throughput is >10K ops/sec for our schema (500-byte order records)"  
❌ **Weak:** "Database is scalable"

### 2. Why This Assumption Matters (Dependency)

**How does architecture depend on this assumption?**

**Example:**
```
If this assumption is FALSE: Team cannot operationalize Kubernetes → deployment becomes a bottleneck → 
we miss 6-month launch window → revenue delay → fallback: use AWS App Runner (serverless containers, 
less operational burden).

Consequence: If we chose Kubernetes architecture and assumption fails, we're forced into expensive 
rewrite (2–3 weeks) in month 5 of project.
```

### 3. Origin of Assumption

**Where did this assumption come from?**

- **Explicit requirement:** Stated by stakeholder or in requirements document
- **Inferred from domain:** Standard industry practice (e.g., "RESTful APIs are standard for microservices")
- **Inferred from constraints:** Derived from stated constraints (e.g., "6-month timeline implies MVVM, not greenfield design")
- **Speculative:** Educated guess based on experience, not grounded in stated constraints
- **Vendor claim:** Third-party states this (e.g., "Stripe SLA is 99.99%")

**Example:**
```
Origin: VENDOR CLAIM
Stripe's published SLA states: "99.99% uptime guarantee for Stripe API"
But our assumption is stronger: "Stripe latency is <200ms p99 under normal load"
The 99.99% only guarantees availability, not latency.
Latency assumption: SPECULATIVE (based on reports from other users, not tested in our context)
```

### 4. Current Confidence Level

**1–5 scale. Justify each level.**

| Level | Meaning | Example Confidence Factors |
|-------|---------|---------------------------|
| 1 | Very uncertain | No prior experience; no evidence; high risk of being wrong |
| 2 | Uncertain | Limited experience; some evidence contradicts assumption |
| 3 | Moderate | Some evidence supports it; but not fully tested in our context |
| 4 | Confident | Evidence supports it; similar projects succeeded; small uncertainty |
| 5 | Near certain | Extensive testing; proven track record; virtually no uncertainty |

**Example:**
```
Assumption: "Our 8-person team can learn Kubernetes in 3 months"

Current Confidence: 2 (UNCERTAIN)

Reasoning:
- Evidence FOR: 2 engineers have done 1 Docker/Kubernetes training workshop
- Evidence AGAINST: No one has operationalized production Kubernetes; 0 production incidents handled
- Timeline risk: 3 months is aggressive; industry standard is 4–6 months for team of this size
- Team capacity: With 6-month launch window, 3 months is month 3 of 5 coding months (very tight)

Confidence = 2: Plausible but needs validation; plan for fallback if this fails early.
```

### 5. Validation Plan

**How will we test if this assumption holds?**

Create concrete milestones with success criteria:

**Example:**

| Milestone | Timeline | Action | Success Criteria | Owner |
|-----------|----------|--------|------------------|-------|
| **Checkpoint 1** | Month 1, week 2 | Complete 1 engineer-week of Kubernetes workshops + proof-of-concept (deploy 1 service to dev cluster) | PoC deployment succeeds; 2 engineers can troubleshoot basic pod crash | Platform Lead |
| **Checkpoint 2** | Month 2, week 1 | Deploy 2–3 core services to staging cluster; simulate 1 incident (pod crash, node failure) | Teams respond to simulated incident in <10 min; root cause identified; fix deployed | SRE Lead |
| **Checkpoint 3** | Month 3, week 2 | Deploy to production; handle 1 real incident from operations team | Real incident resolved in <30 min; confidence improves to 4/5 | On-call Engineer |
| **Final Validation** | Month 3, end | Team assessment: Can team operationalize K8s production env without external help? | 80% of team can deploy, scale, and troubleshoot independently | CTO |

**Fallback Trigger:** If checkpoint 1 fails (PoC deployment stalled >1 week), escalate to fallback plan (AWS App Runner).

### 6. Evidence Threshold (What Confirms/Invalidates?)

**What level of proof would confirm or invalidate this assumption?**

**Example:**
```
Assumption: "Payment Gateway latency <200ms p99"

CONFIRMATION THRESHOLD:
- Run load test: 1,000 concurrent requests to Stripe
- Measure: Stripe API response latency across 24-hour period (daylight + night traffic)
- Accept if: p99 latency <200ms AND p99.9 <300ms AND error rate <0.1%
- Test in our real environment (not Stripe's demo environment)
- Repeat monthly to detect degradation

INVALIDATION THRESHOLD:
- If p99 latency >300ms for any 1-hour period during normal business hours
- OR if error rate >0.5% for >10 minutes
- → Invalidates assumption; trigger contingency plan (circuit breaker, fallback to local caching)
```

### 7. Fallback Plan

**If assumption proves false, what's plan B?**

**Format:**

| If Assumption Fails | Fallback Plan | Cost / Timeline | Operational Impact |
|-------------------|---------------|-----------------|-------------------|
| Team cannot learn Kubernetes in 3 months | Use AWS App Runner (serverless containers); simpler ops model | ~2 weeks rewrite; ~$2K/month higher infrastructure cost | Simpler on-call; less deployment flexibility; possible cost increase if scale >1K req/sec |
| Payment Gateway latency spikes >200ms | Implement client-side caching (60-second TTL); circuit breaker if P Gateway timeout | ~1 week implementation; operational monitoring of circuit breaker | Orders may have stale product prices for 60 sec; acceptable for retail SLA |
| Database write throughput <10K ops/sec | Switch to write replicas + sharding by customer_id; adds operational complexity | ~2 weeks per shard; scales to 100K+ ops/sec if needed | 2–3 new failure modes (shard coordination, cross-shard queries); team training required |

### 8. Validation Status & Timeline

**When do we need this validated?**

**Example:**
```
Assumption: Team can learn Kubernetes in 3 months
Deadline: End of month 1 (checkpoint 1)
Rationale: If learning stalls, fallback requires 2 weeks; we need to know by month 1.5 to stay on track.

Status: VALIDATING
- Checkpoint 1 in progress (week 2 of month 1)
- 2 engineers completed workshop
- PoC deployment underway (expected completion: end of week 2)
- Next update: Week 3 (month 1, day 21)
```

### 9. Review & Escalation

**Quarterly or when validated:**

- **Current Status:** Unvalidated / Validating / Validated / Invalid
- **Updated Evidence:** New data collected since last review
- **Confidence Change:** Previous 2/5 → Current 3/5 (improvement? degradation?)
- **Action:** Continue validation / Activate fallback / Archive (assumption proven correct)

**Escalation Rules:**
- If confidence drops (e.g., from 3/5 → 1/5), page project manager
- If deadline for validation is missed, escalate to CTO
- If fallback plan must be activated, notify all affected teams

### 10. Sign-Off

- **Identified by:** [Name]
- **Validated by:** [Name]
- **Approved by:** [CTO or authority]
- **Validation started:** [YYYY-MM-DD]
- **Expected validation complete:** [YYYY-MM-DD]
- **Next review:** [YYYY-MM-DD]

---

## Quality Checklist

- [ ] Assumption statement is specific and testable (not vague)
- [ ] Why it matters is clear (what breaks if false?)
- [ ] Origin is stated (vendor claim? inferred? speculative?)
- [ ] Confidence level (1–5) has explicit justification
- [ ] Validation plan has concrete milestones with success criteria
- [ ] Evidence threshold defines what counts as confirmed/invalidated
- [ ] Fallback plan is realistic and costed
- [ ] Validation deadline is justified (when do we need to know?)
- [ ] Owner is named with accountability
- [ ] Status is current (not stale)

---

## Anti-Patterns

❌ **Vague assumption:** "Team is capable"  
✓ **Specific assumption:** "Our 8-person team can operationalize Kubernetes production environment in 3 months without external consultants"

❌ **No validation plan:** "We'll figure it out during development"  
✓ **Concrete validation:** "Month 1 checkpoint: PoC deployment to dev cluster; 2 engineers can troubleshoot pod crashes"

❌ **Unrealistic fallback:** "If assumption fails, we'll redesign entire architecture"  
✓ **Realistic fallback:** "If learning stalls by month 2, switch to AWS App Runner; 2-week rewrite; acceptable cost increase"

❌ **No deadline:** "We'll validate this sometime"  
✓ **Explicit deadline:** "Must be validated by end of month 1; if not confirmed, activate fallback plan by month 1.5"

---

## Common Assumption Categories

1. **Skill / Staffing** — Team can learn new technology; person won't leave; hiring timeline
2. **Technical Capability** — Payment Gateway latency; database throughput; third-party API reliability
3. **Vendor SLA** — Cloud provider uptime; third-party API performance; managed service capabilities
4. **Operational Maturity** — Team can operate Kubernetes; oncall team available; runbooks exist
5. **Market / Business** — Customers accept eventual consistency; scale will reach 100K users; revenue model viable
6. **Regulatory** — GDPR applies to our data; compliance audits every 12 months
7. **Timeline** — 6-month launch window is achievable; team productivity remains steady

---

## Example: Complete Assumption Register Entry

**ID:** ASSUME-001  
**Assumption:** Team of 8 engineers can operationalize Kubernetes production in 3 months  
**Category:** Skill / Operational Maturity  
**Owner:** Platform Lead  
**Confidence:** 2/5 (uncertain; needs validation)  
**Dependency:** If false, entire deployment strategy requires rewrite (fallback: App Runner)  
**Origin:** Inferred from constraint "6-month launch window" (assumed MVVM implies learning K8s required)  

**Validation Plan:**
- Month 1, week 2: PoC deployment; 2 engineers troubleshoot pod crash
- Month 2, week 1: Stage deployment; handle simulated incident (<10 min)
- Month 3, week 2: Prod deployment; handle real incident (<30 min)
- Fallback trigger: If PoC deployment delayed >1 week, activate App Runner fallback

**Evidence Threshold:** Confirmation = all 3 checkpoints passed; team independently operates prod. Invalidation = any checkpoint fails or timeline slips >1 week.

**Fallback:** Switch to AWS App Runner; 2-week rewrite; $2K/month higher ops cost; simpler on-call.  
**Validation Deadline:** End of month 1 (checkpoint 1); If not confirmed, fallback by month 2.  
**Status:** VALIDATING (checkpoint 1 in progress; update end of week 2)
