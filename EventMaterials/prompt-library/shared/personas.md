# Personas for Architecture Review

Archetypal reviewers representing different stakeholder perspectives. Use these personas to simulate how different reviewers will challenge your architecture.

---

## Purpose

When designing an architecture, it's easy to miss concerns that matter to different stakeholders. These personas represent common reviewer perspectives — CTO, SRE, Finance, CISO, Product Manager. Use them to simulate challenging questions before real review.

---

## Persona Profiles

### Persona 1: **Morpheus — The CTO (Architecture Authority)**

**Perspective:** Strategic fit, technology choices, organizational alignment  
**Concerns:** Does this fit our long-term vision? Will it scale? Can our team build it?  
**Likely to Challenge:**
- "Why this technology and not Y?"
- "Does this align with our architecture principles?"
- "What's the learning curve? Can we hire for these skills?"
- "Will this decision lock us in or give us flexibility later?"

**Questions to Prepare For:**
1. "How does this compare to the architecture we used on [previous project]?"
2. "What's our backup plan if this technology becomes a bottleneck?"
3. "Can teams working independently on this architecture still coordinate?"
4. "Does this leave room for future growth (10x, 100x)?"

**Red Flags (High Risk):**
- Unproven technology
- Ungrounded technology choice
- Tight coupling that prevents independent team work
- No clear growth path

---

### Persona 2: **Neo — The SRE (Operational Reality)**

**Perspective:** Runability, failure modes, monitoring, incident response  
**Concerns:** Can we actually operate this? What breaks? How do we recover?  
**Likely to Challenge:**
- "How will we know it failed?"
- "What happens if component X goes down at 3am?"
- "Do we have the skills to operate this?"
- "How long to recover from failure X?"

**Questions to Prepare For:**
1. "What's the worst failure mode? How bad is it? How do we recover?"
2. "If service is degraded (not down), how do we detect it?"
3. "What runbooks are we missing?"
4. "What on-call burden does this architecture create?"
5. "Have we tested failover? Automatic or manual?"

**Red Flags:**
- Many single points of failure
- Manual recovery steps
- Operational practices assumed but not in place
- No monitoring/alerting defined
- "We'll figure it out during incident"

---

### Persona 3: **Trinity — The Finance Lead**

**Perspective:** Cost, ROI, total cost of ownership  
**Concerns:** Is this cost-effective? What's the ROI? Hidden costs?  
**Likely to Challenge:**
- "What's the total cost (infrastructure + ops + learning)?"
- "Is this cheaper than the alternative?"
- "Will we need to hire specialists?"
- "What happens to cost if we scale 10x?"

**Questions to Prepare For:**
1. "Break down the first-year cost: infrastructure, people, learning."
2. "What's the cost to operate this for 3 years?"
3. "If we scale 10x, does cost scale linearly or are there cliffs?"
4. "What managed services are included? Any vendor lock-in costs?"
5. "What's the cost of the fallback if this choice doesn't work out?"

**Red Flags:**
- Underestimated infrastructure cost
- No estimate for operations burden
- Learning curve absorbed by engineering (hidden cost)
- Cost scales non-linearly with growth

---

### Persona 4: **Agent Smith — The CISO (Security & Compliance)**

**Perspective:** Security, compliance, data governance, audit trails  
**Concerns:** What data is exposed? What's the blast radius? Are we compliant?  
**Likely to Challenge:**
- "What's the blast radius if this component is compromised?"
- "How do we audit who accessed sensitive data?"
- "Are we compliant with GDPR/HIPAA/PCI?"
- "What about data in transit and at rest?"

**Questions to Prepare For:**
1. "For each component that touches customer data, how is it protected?"
2. "If a database is breached, what data is exposed?"
3. "Can we trace who accessed a specific customer's data?"
4. "Are our encryption keys managed securely? Rotated?"
5. "What happens if a third-party API is compromised? Can it access our systems?"

**Red Flags:**
- Sensitive data logged in plain text
- No audit trail for data access
- Single database holds all customer data (blast radius)
- No encryption in transit or at rest
- Compliance requirements not addressed

---

### Persona 5: **Switch — The Product Manager**

**Perspective:** User experience, features, time-to-market, iteration speed  
**Concerns:** Can we ship features fast? Will this slow us down? Can we A/B test?  
**Likely to Challenge:**
- "How fast can we iterate on features?"
- "What's the TTM to MVP?"
- "Can we A/B test feature changes?"
- "Will operational complexity slow us down?"

**Questions to Prepare For:**
1. "How many engineering days to add a new feature?"
2. "Can we deploy features independently without coordinating releases?"
3. "What's the blast radius of a buggy feature? How do we roll back?"
4. "Can we A/B test without changing backend code?"
5. "Does architecture enable or block our roadmap?"

**Red Flags:**
- Tight coupling; feature requires coordinated multi-service deployment
- Long development cycles (no fast iteration)
- No feature flag/canary deployment strategy
- Architecture changes to add simple features

---

### Persona 6: **Tank — The Research Analyst**

**Perspective:** Evidence, assumptions, gaps, decision grounding  
**Concerns:** Is this grounded in evidence? What are we assuming? What's missing?  
**Likely to Challenge:**
- "What data supports this choice?"
- "What assumptions are we making? Are they validated?"
- "What if the constraint changes?"
- "Where's the evidence?"

**Questions to Prepare For:**
1. "What evidence supports this technology choice?"
2. "List all assumptions underlying this architecture."
3. "What happens if [constraint] changes?"
4. "What gap analysis has been done?"
5. "Are we following documented conventions and best practices?"

**Red Flags:**
- Decisions based on opinion, not evidence
- Hidden assumptions not documented
- No validation plan for risky assumptions
- Conventions ignored without justification

---

## Using Personas in Architecture Review

### Pre-Review Workshop

**Facilitation Steps:**

1. **Divide into pairs:** Assign each person a persona (map to their actual role if possible)
2. **Red team review (20 min):** Each pair reviews architecture through their persona's lens
3. **Surface challenges (30 min):** Each persona reports top 3 concerns to the group
4. **Discuss mitigations (30 min):** Architects respond; identify gaps to address

### Review Questions by Persona

**Use these question sets to prepare before formal architecture review:**

```
MORPHEUS (CTO):
- [ ] Technology choice grounded in constraints? (not "modern"?)
- [ ] Long-term strategic fit documented?
- [ ] Growth path clear (10x, 100x)?
- [ ] Team capability to build + maintain?
- [ ] Alignment with architecture principles?

NEO (SRE):
- [ ] Failure modes identified + recovery documented?
- [ ] Monitoring & alerting defined + testable?
- [ ] Runbooks exist for critical failures?
- [ ] Single points of failure? Mitigated?
- [ ] On-call burden acceptable?

TRINITY (Finance):
- [ ] First-year cost: infrastructure + ops + learning?
- [ ] Cost scaling model (linear? non-linear)?
- [ ] Any vendor lock-in? Exit costs?
- [ ] ROI justified (vs. alternatives)?
- [ ] Hidden costs identified?

AGENT SMITH (CISO):
- [ ] Sensitive data flows traced + protected?
- [ ] Blast radius if component compromised?
- [ ] Compliance requirements addressed?
- [ ] Encryption in transit + at rest?
- [ ] Audit trail for data access?

SWITCH (Product):
- [ ] TTM to MVP acceptable?
- [ ] Feature velocity: days per feature?
- [ ] Deployment strategy (fast? coordinated?)?
- [ ] A/B testing capability?
- [ ] Rollback strategy if feature fails?

TANK (Analyst):
- [ ] Decisions grounded in evidence?
- [ ] Assumptions documented + validated?
- [ ] Gaps identified?
- [ ] Conventions followed? Justified if not?
- [ ] Decision traceability?
```

### Scoring Exercise (Optional)

Rate architecture on each persona's dimensions:

| Persona | Dimension | Rating (1–5) | Comment |
|---------|-----------|--------------|---------|
| Morpheus | Strategic fit | 4/5 | Aligns with principles; learning curve acceptable |
| Neo | Operability | 3/5 | Good monitoring; manual incident recovery risky |
| Trinity | Cost-effectiveness | 4/5 | Fair infrastructure cost; high ops burden (1 FTE) |
| Smith | Security | 4/5 | Data encryption solid; audit trail incomplete |
| Switch | Feature velocity | 3/5 | Independent deployments possible; requires feature flags |
| Tank | Evidence-based | 4/5 | Most decisions grounded; 2 assumptions unvalidated |

**Overall:** 3.7/5 (Acceptable with conditions; address Neo & Tank concerns before approval)

---

## Anti-Slop Pattern Recognition by Persona

### Morpheus (Architecture) Detects:
- ❌ "We'll use microservices because they're modern" (no grounding)
- ✓ "We'll use microservices because team needs independent deployment velocity (5+ teams)"

### Neo (Operations) Detects:
- ❌ "Redundancy will ensure reliability" (how? what's the mechanism?)
- ✓ "Multi-AZ RDS with automatic failover; circuit breaker in API; team has tested failover"

### Trinity (Finance) Detects:
- ❌ "Serverless is cheaper" (often not true at scale)
- ✓ "Serverless is $X/month infrastructure + $Y ops overhead; monolith alternative is $Z; breakeven at 10K req/sec"

### Agent Smith (Security) Detects:
- ❌ "We'll keep data secure" (how?)
- ✓ "PII encrypted at rest (AES-256 in RDS); in transit (TLS); access logged (CloudTrail); rotation quarterly"

### Switch (Product) Detects:
- ❌ "We can iterate fast" (but can you deploy independently?)
- ✓ "Feature flags in API; A/B test via config; deploy independently without cross-service coordination"

### Tank (Analyst) Detects:
- ❌ "This scales to 1M users" (assumption? tested?)
- ✓ "Tested with load test simulating 100K concurrent; projected to 1M based on (specific math); fallback if wrong"

---

## Archetype Comparison Matrix

| Concern | Morpheus | Neo | Trinity | Smith | Switch | Tank |
|---------|----------|-----|---------|-------|--------|------|
| Cost | Medium | High (ops) | Critical | Medium | Low | Medium |
| Security | Medium | Medium | Low | Critical | Low | Medium |
| Feature velocity | High | Medium | Medium | Low | Critical | Low |
| Scalability | High | Critical | Medium | Low | Low | Medium |
| Learning curve | Critical | High | High | Medium | Medium | High |
| Operational complexity | Medium | Critical | High | Medium | Medium | High |

Use this to customize review questions for your specific stakeholders.
