# Confidence Scoring Template

Template for rating confidence levels on architecture decisions, assumptions, and risk mitigation.

---

## Purpose

Confidence scoring makes uncertainty explicit. Instead of pretending everything is certain, document what you're confident about and what's speculative. This enables risk management.

---

## Confidence Scale (1–5)

### Level 5: Near Certain
**Definition:** Proven track record; extensive testing; virtually no uncertainty  
**Indicators:**
- Used successfully in similar projects (3+ examples)
- Tested in our specific context
- No contradicting evidence
- Team has deep expertise

**Example:** "PostgreSQL can handle 10K writes/sec for our schema" (confidence 5)
- Evidence: Load tested with production-like data; sustained 12K writes/sec for 24 hours
- Team has 5+ years PostgreSQL experience
- No concerns identified

### Level 4: Confident
**Definition:** Evidence supports it; small uncertainty remains  
**Indicators:**
- Published case studies or academic research support it
- Partial testing in similar context
- Rare contradicting evidence
- Team has moderate expertise

**Example:** "Team can learn Kubernetes in 3 months" (confidence 4)
- Evidence: Similar teams did it in 3.5 months; our team has Docker experience
- Small uncertainty: Learning curve depends on team motivation; one person might take longer

### Level 3: Moderate
**Definition:** Some evidence supports it; significant uncertainty exists  
**Indicators:**
- Limited testing; only in different context
- Some contradicting evidence
- Assumptions required
- Team has limited expertise

**Example:** "Payment Gateway latency <200ms p99" (confidence 3)
- Evidence: Stripe SLA is 99.99% (availability, not latency); one report from similar user measured <300ms
- Uncertainty: Our network conditions might be different; vendor claims often optimistic

### Level 2: Uncertain
**Definition:** Limited evidence; significant risk if wrong  
**Indicators:**
- Few examples; mostly anecdotal
- Contradicting evidence exists
- Many assumptions required
- Team has little/no expertise

**Example:** "We can operationalize multi-region disaster recovery without specialists" (confidence 2)
- Evidence: None; team has no multi-region experience
- Significant uncertainty: Multi-region DR is complex; many failure modes not understood

### Level 1: Very Uncertain
**Definition:** Virtually no evidence; high risk if wrong  
**Indicators:**
- No examples; purely speculative
- Contradicting evidence
- Many unvalidated assumptions
- Team has zero expertise; new technology

**Example:** "We can deploy to edge nodes globally within 12 months" (confidence 1)
- Evidence: None; never attempted before
- Extremely speculative; many unknowns in edge deployment, security, cost

---

## Confidence Scoring Worksheet

**For each architecture decision, estimate confidence:**

### Scenario 1: Database Choice

**Claim:** "PostgreSQL is the right database for our order system"

| Factor | Confidence | Reasoning |
|--------|-----------|-----------|
| **Requirement fit** | 5/5 | ACID transactions explicitly required; PostgreSQL proven for this |
| **Scale suitability** | 4/5 | Tested to 100K rows; projected 1M rows over 12 months; some indexing tuning expected |
| **Team expertise** | 5/5 | 4 of 8 engineers have 5+ years PostgreSQL experience |
| **Operational burden** | 4/5 | Team can manage; backup/recovery procedures needed (haven't done yet) |
| **Cost** | 5/5 | RDS pricing clear; predictable cost model |

**Overall Confidence: 4.4/5 (Confident)**  
**Lowest scoring factor:** Operational burden (team hasn't managed backups yet)  
**Risk:** If backup/recovery procedures fail during incident, confidence drops to 2/5  
**Mitigation:** Practice backup/recovery procedure in Q1

---

### Scenario 2: Microservices Adoption

**Claim:** "We can implement microservices architecture with our 8-person team in 6 months"

| Factor | Confidence | Reasoning |
|--------|-----------|-----------|
| **Requirement fit** | 3/5 | Team needs independent deployments (yes); but team has no microservices experience |
| **Team expertise** | 2/5 | Microservices is new skill for team; 1 engineer has read books, none have production experience |
| **Learning curve** | 2/5 | Estimate: 2–3 months learning; 3–4 months implementation; tight for 6-month timeline |
| **Operational burden** | 2/5 | Debugging distributed failures is hard; team not ready for on-call complexity |
| **Scale suitability** | 4/5 | Microservices scales well; but overkill for initial scale (1K concurrent users) |

**Overall Confidence: 2.6/5 (Uncertain)**  
**Mitigations Required:**
1. Reduce risk by starting with monolith; plan microservices rewrite for year 2
2. Or: Hire microservices specialist by month 2
3. Or: Reduce team learning time by starting monolith + learning during month 1

**Decision:** Start monolith (confidence 5/5); plan microservices for year 2

---

## Using Confidence Scores in Decisions

### Decision Filter

**For each architecture option, calculate overall confidence:**

| Option | Confidence Score | Decision Impact |
|--------|------------------|-----------------|
| Monolith + PostgreSQL | 4.5/5 (confident) | ✓ RECOMMENDED |
| Microservices + gRPC | 2.6/5 (uncertain) | ⚠️ HIGH RISK; requires mitigation |
| Serverless + DynamoDB | 3.2/5 (moderate) | ⚠️ ACCEPTABLE if budget permits learning |

**Decision Rule:** Choose option with highest confidence (ideally >4/5) OR choose lower confidence if risk is mitigated.

---

## Confidence Tracking Over Time

**Update confidence scores as you learn:**

| Date | Factor | Old Confidence | New Confidence | Evidence | Change |
|------|--------|-----------------|-----------------|----------|--------|
| Day 1 | Team learns Kubernetes | 2/5 | 2/5 | No change yet | — |
| Week 2 | Team learns Kubernetes | 2/5 | 3/5 | PoC deployment succeeded | +1 |
| Week 4 | Team learns Kubernetes | 3/5 | 3/5 | Handling simple pod crash; took 20 min | 0 |
| Week 8 | Team learns Kubernetes | 3/5 | 4/5 | Handled production incident; 15 min MTTR | +1 |

**Dashboard:** Track confidence scores over project lifecycle; update when new evidence appears.

---

## Confidence-Based Risk Management

### High Confidence (5/5)
**Action:** Proceed with minimal oversight. Document decision and move on.

### Confident (4/5)
**Action:** Proceed. Monitor the uncertain factor. Plan checkpoint if confidence could drop.

### Moderate (3/5)
**Action:** Proceed with caution. Identify validation milestone. If milestone fails, escalate.  
**Example:** "If learning Kubernetes takes >6 weeks, we'll switch to App Runner" (set checkpoint at week 6)

### Uncertain (2/5)
**Action:** Proceed only if high upside and mitigation exists.  
**Example:** "Microservices is risky (2/5) but necessary for team independence. Mitigation: hire specialist by month 2. If not available, revert to monolith."

### Very Uncertain (1/5)
**Action:** DO NOT PROCEED without solving uncertainty first.  
**Example:** "Edge deployment (1/5 confidence) — don't commit to it. Run spike to explore feasibility first (1–2 weeks). Then reassess confidence."

---

## Confidence Scoring by Role

### Morpheus (Lead Architect)
**Scores:** Overall architecture fit, long-term viability, strategic alignment  
**Scoring Rule:** "Does this align with our principles? Will it serve our roadmap?"

### Neo (SRE)
**Scores:** Operability, failure mode recovery, monitoring, incident response  
**Scoring Rule:** "Can we actually run this? What goes wrong? How do we fix it?"

### Trinity (Finance)
**Scores:** Cost estimation accuracy, budget fit, ROI  
**Scoring Rule:** "Is this cost realistic? Will we stay within budget?"

### Tank (Analyst)
**Scores:** Evidence quality, assumption validation, decision grounding  
**Scoring Rule:** "Is this decision backed by evidence or speculation?"

### Switch (Product)
**Scores:** Feature velocity, deployment independence, rollback capability  
**Scoring Rule:** "Can we ship features fast? Will we slow down?"

### Agent Smith (Security)
**Scores:** Security architecture, compliance fit, data protection  
**Scoring Rule:** "Are we secure? Are we compliant?"

---

## Confidence Communication Template

**When presenting architecture to stakeholders:**

```
"ARCHITECTURE DECISION: Monolith + PostgreSQL + Redis

OVERALL CONFIDENCE: 4.5/5 (Confident)

High-Confidence Factors (5/5):
- Database choice: ACID required; PostgreSQL proven; team expertise available

Moderate-Confidence Factors (3–4/5):
- Team learning: Operational procedures needed; team has not done backups
- Scaling trajectory: 1K → 10K users confident; 10K → 100K uncertain (may need rework)

Uncertain Factors (2/5):
- Multi-region disaster recovery: Team has no experience; defer to year 2

MITIGATIONS:
- Backup/recovery procedure: Practice in Q1 (improves confidence to 5/5)
- Scaling plan: Monitor growth metrics; reassess at 5K users; document sharding strategy

FALLBACK:
- If scaling to 100K users fails with monolith: Rewrite to microservices (2–4 week effort)
- If team can't handle operational burden: Hire SRE specialist by month 6

DECISION: Proceed with monolith. Monitor confidence factors at checkpoints.
```

---

## Anti-Patterns

❌ **False confidence:** "We're 99.9% confident this will work" (what evidence? too high for most decisions)  
✓ **Calibrated confidence:** "We're 4/5 confident; here's the evidence and what could cause us to be wrong"

❌ **Unstated assumptions:** "This is simple" (simple for whom? based on what?)  
✓ **Explicit assumptions:** "This is simple (3/5) IF team has Docker experience; if not, learning curve adds 2 weeks"

❌ **Overconfidence in novel situations:** "We've never done this, but it'll be fine" (1/5 confidence disguised as 5/5)  
✓ **Honest about novelty:** "This is new for us (2/5 confidence); we'll run a spike to assess feasibility"

---

## Confidence Score Calibration Exercise

**Rate these claims on 1–5 scale; then compare with team:**

1. "Our team can deploy to Kubernetes production in 3 months" — Your score: __/5
2. "Payment Gateway latency will be <200ms p99" — Your score: __/5
3. "Database will handle 10x growth without rewrite" — Your score: __/5
4. "Feature deployment will take <15 minutes end-to-end" — Your score: __/5
5. "Multi-region disaster recovery will work seamlessly" — Your score: __/5

**Team discussion:** Why did scores differ? What evidence would move your confidence up or down?

---

## Confidence Checkpoints

**Build checkpoints into project to reassess confidence:**

| Checkpoint | What to Reassess | Trigger Re-score If |
|-----------|-----------------|-------------------|
| End of Month 1 | Team learning (Kubernetes, event-driven patterns) | Actual learning slower than projected |
| End of Month 2 | Scaling performance (database, caching effectiveness) | Performance metrics show bottleneck |
| End of Month 3 | Operational procedures (incident response, recovery) | First production incident reveals gaps |
| End of Month 6 | Overall architecture (still fits, or needs rework?) | Major constraint changed or new learnings |

**Action:** If confidence drops >1 point, escalate and consider mitigation/fallback.
