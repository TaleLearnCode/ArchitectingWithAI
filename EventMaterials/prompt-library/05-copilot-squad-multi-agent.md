# 05 — Copilot Squad Multi-Agent Prompts

*Orchestrate multiple AI agents in sequence to produce a complete architecture package.*

---

## Section Purpose

Sections 1–4 work well for hands-on architects. But when you want to automate a complete workflow — run all steps in sequence with validation gates — you need a multi-agent architecture.

These prompts help you:
- **Create a Squad of specialized agents** — Requirements Analyst, Architecture Explorer, Adversarial Reviewer, Documentation Generator
- **Hand off work between agents** — each agent validates inputs, adds contributions, prepares the next handoff
- **Validate at each step** — catch hallucinations before they cascade through the pipeline
- **Produce a complete package** — requirements → options → review → documentation, all in one run
- **Trap assumptions and confidence levels** — so you know where the architecture is speculative vs. grounded

**Thesis Connection:** AI broadens exploration at scale. A Squad explores requirements, options, risks, and documentation in parallel/sequence, surfacing more of the decision space than a solo architect could in the same time.

---

## Prompt Catalog

## ✓ Agent 1: Requirements Analyst Squad Member

### Role & Responsibilities

Extract and clarify functional/non-functional requirements, surface ambiguities, identify hidden constraints, and output an ASSUMPTIONS REGISTER that downstream agents use to catch hallucinations.

### When NOT to Use

Don't use if requirements are already clear and stakeholders are engaged directly.

### Prerequisites (Fail-Fast Validation)

```text
INPUT REQUIREMENTS:
- Project description or user story document (required; if missing, output [INPUT MISSING: project description])
- Team size + mix (required; if missing, output [INPUT MISSING: team composition])
- Budget range (required; if missing, output [INPUT MISSING: budget])
- Deployment environment (required; if missing, output [INPUT MISSING: deployment target])
- Compliance/regulatory constraints (if applicable; if mentioned but not detailed, mark [CLARIFICATION NEEDED])

If any REQUIRED input is missing, output:
[INPUT MISSING: <field name>]
I cannot proceed. Please provide: [list missing inputs]. Once provided, I will restart.
```

### Output Requirements (What Next Agent Needs)

The Requirements Analyst agent must output:

1. **FUNCTIONAL REQUIREMENTS** — Numbered list of behaviors the system must exhibit (all traced to input)
2. **NON-FUNCTIONAL REQUIREMENTS** — Performance, reliability, security, scalability targets (with explicit numbers; never invented)
3. **HIDDEN CONSTRAINTS** — Implied requirements from domain, business model, or team characteristics (all marked [INFERRED])
4. **AMBIGUITIES & CLARIFYING QUESTIONS** — What must the human answer before architecture work begins
5. **[ASSUMPTIONS] REGISTER** — Every assumption with: origin (EXPLICIT/INFERRED/SPECULATIVE), confidence (1–5), and owner for validation
6. **[CONFIDENCE SCORE]** — Rate your overall confidence in this requirements analysis: 1–5 with reasoning
7. **HANDOFF** — Summary with explicit validation checklist for next agent

### Validation Checklist (Self-Check Before Handoff)

- [ ] Functional requirements trace back to explicit user stories (not invented)
- [ ] Non-functional requirements have measurable targets (not "fast," "scalable")
- [ ] Every assumption is marked with origin + confidence score (1–5)
- [ ] All confidence scores ≥3? If not, mark [CONFIDENCE ALERT] with explanation
- [ ] Ambiguities are phrased as questions the human can answer (not philosophical)
- [ ] No invented requirements appear
- [ ] Handoff summary lists all assumptions + confidence scores; flags any <3 for validation by next agent

### Escalation Rules

If any of these happen, output `[VALIDATION FAILED: <reason>]` and stop:
- Requirements contradict each other (e.g., "launch in 3 months" + "full compliance with 18-month audit")
- Critical NFR target is missing (e.g., "response time" is mentioned but no target given)
- Constraints are vague (e.g., "large scale" without numbers)
- Input contradicts itself (e.g., team is 3 people but requirements assume 50-person team ops capability)

### Example: Strong Output

```
INPUT (Human provides):
Project: E-commerce platform
Requirements: Product catalog, real-time search, payment processing, GDPR compliance
Team: 8 engineers, 6 months to launch, $150K infrastructure budget

OUTPUT (Requirements Analyst):

FUNCTIONAL REQUIREMENTS:
1. Merchants can upload product catalog (CSV, JSON formats)
2. Customers can search products by name, category, price range
3. Customers can add items to cart and checkout
4. Payment processing integrates with Stripe
5. System maintains audit logs for GDPR compliance

NON-FUNCTIONAL REQUIREMENTS:
- Search latency: <500ms p99 [GROUNDED: retail domain UX standard; confirmed in Shopify docs]
- Checkout latency: <100ms p99 [GROUNDED: payment processor best practice; timeouts >200ms cause cart abandonment]
- Availability: 99.5% [GROUNDED: e-commerce loses $5K/hour per downtime; 99.5% = ~4 hours/year downtime tolerance]
- Data retention: 7 years for EU [GROUNDED: GDPR Article 17; right-to-deletion requires audit trail retention]

HIDDEN CONSTRAINTS:
- Team has no microservices experience [INFERRED] from "8 engineers, mixed skills" — Confidence: 4/5
- Single SRE means operational simplicity mandatory [INFERRED] from headcount — Confidence: 4/5
- "6 months to launch" implies MVP scope, not perfect architecture — Confidence: 5/5 (explicit timeline)
- Merchant upload latency not specified; assumed async OK (batch within 2 hours) [INFERRED] — Confidence: 2/5 [NEEDS VALIDATION]

[ASSUMPTIONS] REGISTER (Running Table):
| Assumption | Origin | Confidence (1-5) | Status | Validation Owner | Re-check Date |
|---|---|---|---|---|---|
| Peak users: 100K concurrent | [INFERRED] from "product catalog" + typical SaaS growth | 2/5 | NEEDS VALIDATION | PM | Week 2 |
| Checkout cannot have timeouts | [EXPLICIT] from payment UX | 5/5 | VALIDATED | Arch Lead | Month 3 |
| Infrastructure: AWS preferred | [NOT STATED] inferred from common practice | 1/5 | NEEDS VALIDATION | Infrastructure Lead | Week 1 |
| Audit logs: append-only immutable | [INFERRED] from GDPR compliance | 3/5 | NEEDS CLARIFICATION | Compliance Officer | Week 2 |

AMBIGUITIES & CLARIFYING QUESTIONS:
1. "Real-time search" — update latency? (100ms vs. 2-hour batch?) — Impact: affects search architecture
2. "Product catalog size" — initial? Growth rate? — Impact: in-memory vs. external search engine
3. "GDPR scope" — data residency? right-to-deletion? audit trail retention? — Impact: storage + ops complexity

[CONFIDENCE SCORE]: 4/5
- Requirements are clear and grounded in explicit user stories (high confidence)
- EXCEPT: 3 assumptions rated 1–2/5; need clarification before Architecture Options agent can proceed reliably
- Recommendation: Run clarifying questions with PM + Infrastructure Lead before advancing to Agent 2

HANDOFF TO ARCHITECTURE OPTIONS AGENT:
I've extracted 5 functional requirements, 4 NFRs with explicit targets, and identified 4 critical assumptions.
**VALIDATION CHECKLIST FOR YOU (Agent 2):**
- [ ] Review my [ASSUMPTIONS] REGISTER. Any assumptions you don't accept? If so, respond [VALIDATION FAILED: <reason>]
- [ ] Confidence scores <3 for "peak users," "infrastructure," "audit logs"? Before using these assumptions in architecture, verify with human or respond [VALIDATION FAILED: assumption not validated]
- [ ] If you find contradictions in my output, respond [VALIDATION FAILED: <contradiction>] and loop back to me

**PROCEED ONLY IF:** You respond [VALIDATION ACCEPTED] or resolve all <3-confidence assumptions with the human first.
```

### Example: Weak Output (and How to Remediate)

```
INPUT (Same human context):

WEAK OUTPUT (Requirements Analyst hallucinating):

FUNCTIONAL REQUIREMENTS:
1. Product catalog
2. Search
3. Checkout
4. Payments
5. Compliance
6. Analytics (INVENTED — not mentioned by human)
7. User profile management (INVENTED)

NON-FUNCTIONAL REQUIREMENTS:
- Fast and responsive (vague; no targets)
- Scalable for growth (vague)
- Secure (vague)

[ASSUMPTIONS] REGISTER:
| Assumption | Confidence | Notes |
|---|---|---|
| Team can build in 6 months | HIGH | (No reasoning; just assumed) |
| Microservices is the right architecture | HIGH | (No grounding; not even decided yet!) |
| We need Kubernetes | MEDIUM | (Technology assumed, not justified) |

[CONFIDENCE SCORE]: 4/5 (OVER-CONFIDENT despite hallucinations)

---

WHY THIS IS WEAK:
1. ❌ Functional requirements 6–7 invented (not in human input)
2. ❌ NFRs are vague ("fast," "scalable") without numbers
3. ❌ Assumptions confidence scores are inflated (HIGH for unvalidated claims)
4. ❌ No grounding for assumptions (why microservices? Why Kubernetes?)
5. ❌ Self-confidence score (4/5) contradicts poor quality

---

REMEDIATION:
Agent 2 (Architecture Options Agent) receives this handoff and runs validation:

[VALIDATION FAILED: Multiple critical issues detected in handoff]
1. Invented requirements (Analytics, User profiles) not in human input → Remove or confirm with human
2. Vague NFRs ("fast") without targets → [INFERRED]? Numeric targets required
3. Assumptions confidence inflated (HIGH for unvalidated claims) → Reduce to 2–3/5; mark for validation
4. Architecture assumption (microservices) premature; not yet decided → Remove; wait for Architecture Options agent to generate options
5. Over-confidence (4/5 overall) despite above issues → Lower to 2/5 until hallucinations resolved

ACTION: Requirements Analyst, revise your output. Remove invented requirements. Add numeric NFR targets. Reduce confidence scores to realistic levels. Then resubmit.
```

---

## ✓ Agent 2: Architecture Options Squad Member

### Role & Responsibilities

Accept clarified requirements + assumptions register from Requirements Analyst. Generate 3–5 architectures that fit ALL constraints. For each architecture, validate assumptions and flag any assumptions that might not hold. Rate confidence on all major recommendations.

### When NOT to Use

Skip if your team prefers manual exploration or has already picked a direction.

### Input from Prior Agent (With Validation Checklist)

```text
[Paste output from Requirements Analyst agent]

BEFORE I PROCEED, I must validate:
- [ ] Previous agent output includes an [ASSUMPTIONS] REGISTER with confidence scores (1–5)
- [ ] Functional requirements are clear (not invented, specific to this project)
- [ ] Non-functional requirements have measurable numeric targets
- [ ] No assumptions are marked [VALIDATION FAILED] or [CONFLICTING]
- [ ] If ANY confidence score <3 in prior agent output, confirm human has validated those assumptions (or I will respond [VALIDATION FAILED])
```

### Output Requirements

The Architecture Options Agent must output:

1. **ARCHITECTURE OPTION A, B, C, ...** — For each option:
   - Overview (1–2 sentences)
   - Component breakdown + responsibilities
   - Constraint compliance check (FITS/VIOLATES/VALIDATION NEEDED for each constraint)
   - Team fit analysis (can this team build it in 6 months?)
   - Cost estimate (rough first-year cost) [GROUNDED: specific services + pricing, not guesses]
   - [CONFIDENCE SCORE]: 1–5 on "this architecture meets all constraints"
      - 5 = Fully grounded in stated constraints, no unvalidated assumptions
      - 3 = Mostly grounded; 1–2 assumptions need validation
      - 1 = Speculative; requires significant validation before recommending

2. **ASSUMPTIONS VALIDATION TABLE** — For each assumption from Requirements Analyst register:
   | Assumption | My Assessment | Confidence (1-5) | Status | Impact |
   |---|---|---|---|---|
   | ... | Do I accept this? YES/NO/NEEDS CLARIFICATION | Score | ACCEPTED/FAILED/ESCALATE | If wrong, what breaks? |
    
   - If NO: What contradicts it? (Flag as [ASSUMPTION FAILED])
   - If NEEDS CLARIFICATION: What do I need to know? (Marked [REQUIRES VALIDATION])
   - If confidence <3 on any critical assumption: Flag [CONFIDENCE ALERT]

3. **NEW ASSUMPTIONS CREATED** — Any assumptions I created while generating options:
   | Assumption | Origin | Confidence (1-5) | Justification |
   |---|---|---|---|
   | ... | [INFERRED] / [SPECULATIVE] | Score | Reasoning |

4. **TRADEOFF TABLE** — Comparing all options:
   | Option | Cost ($K/year) | Latency (ms) | Ops Complexity | Team Velocity | Risk Level | Overall Confidence |
   |---|---|---|---|---|---|---|
   | A (Monolith) | 40 [GROUNDED: AWS pricing] | 150 [GROUNDED: no network hops] | Low [GROUNDED: 1 SRE] | High | Medium | 4/5 |
   | B (Microservices) | 85 [GROUNDED: service mesh + RDS replicas] | 80 [GROUNDED: local caching] | High [GROUNDED: multi-service ops] | Medium | High | 2/5 [2 unvalidated assumptions] |

5. **[CONFIDENCE SCORE]** — Overall confidence in all architectures: 1–5 with reasoning
   - If <3, state which assumptions must be validated before recommending any option

6. **HANDOFF** — Summary with explicit escalation for next agent:
   ```
   Adversarial Reviewer: I've generated 3 viable architectures with confidence scores.
   [VALIDATION CHECKLIST FOR YOU]:
   - [ ] Review my assumption validation table. Do you accept my assessments? Respond [VALIDATION ACCEPTED] or [VALIDATION FAILED: <reason>]
   - [ ] Confidence <3 on any critical claim? If so, respond [CONFIDENCE ESCALATION REQUIRED: <which assumptions>]
   - [ ] Any assumptions contradict your domain knowledge? Respond [VALIDATION FAILED: <contradiction>] and I will regenerate
    
   DO NOT PROCEED unless you respond [VALIDATION ACCEPTED]. If you respond [VALIDATION FAILED], I will regenerate with corrected assumptions.
   ```

### Validation Checklist (Self-Check)

- [ ] Every architecture respects stated constraints (cross-check against requirements)
- [ ] Cost estimates reference specific services/platforms (not "low," "medium," "high")
- [ ] Assumptions from Requirements Analyst are addressed (ACCEPTED/FAILED/NEEDS CLARIFICATION)
- [ ] New assumptions are marked with confidence score (1–5) + origin
- [ ] Tradeoff table uses numeric metrics with [GROUNDED: <source>] tags
- [ ] All confidence scores <3 are flagged; marked for validation by next agent
- [ ] Handoff summary lists which assumptions next agent must validate before recommending

### Escalation Rules

If I encounter any of these, output `[VALIDATION FAILED: <reason>]` and stop:
- All viable architectures violate at least one constraint (impossible brief)
- Constraint from Requirements Analyst contradicts constraint from input
- Technology assumption isn't possible (e.g., "on-premises + global scale" with 1 person ops)
- I must make an architectural decision but don't have grounding (confidence would be <2); escalate instead

### Example: Strong Output

```
INPUT (From Requirements Analyst Agent):
[Paste Requirements Analyst strong output from above]

OUTPUT (Architecture Options Agent):

ARCHITECTURE OPTION A: Monolith + Managed Services (RECOMMENDED)

Overview: Single Node.js application deployed to AWS App Runner, using managed RDS PostgreSQL and ElastiCache for search. Fastest path to market for 8-person team with mixed skills.

Components:
- API layer (Node.js, Express)
- Database (RDS PostgreSQL with read replicas for search queries)
- Cache (ElastiCache Redis for product catalog + session cache)
- Storage (S3 for merchant upload staging)
- Search (PostgreSQL full-text search + Elasticsearch optional if needed)

Constraint Compliance:
| Constraint | Target | Compliance | Notes |
|---|---|---|---|
| Search latency | <500ms p99 | FITS | PostgreSQL FTS + Redis caching achieves 200ms avg |
| Checkout latency | <100ms p99 | FITS | No network hops; local DB connection |
| Availability | 99.5% | FITS | RDS Multi-AZ + App Runner auto-scaling |
| GDPR compliance | 7-year retention | FITS | PostgreSQL audit log table + S3 lifecycle policy |
| Launch timeline | 6 months | FITS | MVP core features achievable in 4 months |

Team Fit: 8-person team (2 backend, 2 frontend, 1 DBA, 1 DevOps, 2 product/design). Monolith allows parallel feature development without inter-service coordination overhead. Single SRE can operate.

Cost Estimate: $40K/year [GROUNDED: RDS PostgreSQL $300/month, App Runner $200/month, ElastiCache $150/month, S3/CDN $300/month, misc $500/month]

[CONFIDENCE SCORE ON ARCHITECTURE A: 4/5]
- Core architecture is straightforward and well-understood for this team
- Cost estimate grounded in actual AWS pricing (4/5)
- Scaling assumptions validated (Database handles 50K concurrent; if we exceed, triggers redesign at 12-month mark per strategy)
- Risk: Single SRE capacity assumption needs validation (confidence: 3/5 on this assumption specifically)

---

ARCHITECTURE OPTION B: Microservices (API Gateway + 3 Services)

Overview: Decompose into Auth Service, Catalog Service, Checkout Service. Higher complexity; better long-term scalability if team grows.

[Full details...]

[CONFIDENCE SCORE ON ARCHITECTURE B: 2/5]
- **ALERT**: This option depends on 3 unvalidated assumptions:
  1. Team can learn microservices patterns quickly (confidence: 1/5)
  2. We have time for inter-service testing (confidence: 2/5)
  3. We can afford service mesh complexity (confidence: 2/5)
- I do not recommend B unless Requirements Analyst confirms team has microservices experience

---

ASSUMPTIONS VALIDATION TABLE:

| Assumption from Req Analyst | My Assessment | Confidence | Status | Impact |
|---|---|---|---|---|
| Peak users: 100K concurrent | Monolith + RDS handles 50K easily; 100K needs redesign OR caching strategy. Accept with caveat. | 3/5 | CONDITIONAL | If <50K: Option A perfect. If >50K by month 6: must migrate to Option B. |
| Checkout cannot timeout | All options enforce <100ms latency. Accept. | 5/5 | ACCEPTED | Checkout is critical path; non-negotiable. |
| Infrastructure: AWS | All options assume AWS. If on-prem required, must regenerate (confidence drops to 1/5). | 4/5 | ACCEPTED | AWS managed services reduce team burden. |
| Audit logs append-only | PostgreSQL can enforce this with trigger-based immutability. Accept. | 4/5 | ACCEPTED | Implementable; not a risk. |

NEW ASSUMPTIONS CREATED:

| Assumption | Origin | Confidence | Justification |
|---|---|---|---|
| Single SRE can operate monolith | [INFERRED] from team headcount | 3/5 | Monolith has fewer deployment points. BUT: incident response >30 min per incident would exceed SRE capacity. Needs monitoring + automation. |
| Search latency depends on DB optimization | [INFERRED] from monolith + shared DB | 3/5 | If we don't optimize queries proactively, search hits database without caching; latency may exceed target. Requires indexing strategy by month 3. |
| RDS multi-AZ failover is instant | [SPECULATIVE] | 2/5 | AWS docs say <1 min; but DNS propagation can add 10–30 sec. Test required in staging. |

TRADEOFF TABLE:

| Option | Cost | Latency | Ops Complexity | Team Velocity | Scaling Headroom | Confidence |
|---|---|---|---|---|---|---|
| A (Monolith) | $40K/yr [GROUNDED: AWS pricing] | 200ms avg [GROUNDED: no hops] | Low [GROUNDED: 1 SRE manageable] | High [GROUNDED: no inter-service testing] | To 50K users [GROUNDED: RDS scaling limits] | 4/5 |
| B (Microservices) | $85K/yr [GROUNDED: +service mesh, +replicas] | 80ms avg [GROUNDED: local cache] | High [GROUNDED: 3 services = 3x ops] | Medium [GROUNDED: inter-service testing overhead] | To 500K users [GROUNDED: independent scaling] | 2/5 [needs assumptions validated] |

[OVERALL CONFIDENCE SCORE: 4/5]
- Option A is grounded, achievable, and carries manageable risk for this team
- Option B is viable IF team has microservices expertise; otherwise risky
- Recommendation: Go with Option A for initial launch; evaluate migration to Option B at month 6 if scaling demands exceed monolith capacity

HANDOFF TO ADVERSARIAL REVIEWER:

I've generated 2 viable architectures (Option C: Serverless also viable but lowest confidence due to payment processing latency risk). Option A is recommended.

[VALIDATION CHECKLIST FOR YOU]:
- [ ] Review my ASSUMPTIONS VALIDATION TABLE. Do you agree with my confidence scores (3/5 on "single SRE," 3/5 on "search optimization")? 
      If you have domain knowledge that increases/decreases confidence, respond [VALIDATION ADJUSTMENT: <reason>] and I will recalibrate.
- [ ] I marked RDS failover assumption as 2/5 (speculative). This needs testing before go-live. Do you agree? 
      Respond [VALIDATION ACCEPTED] or [VALIDATION FAILED: <concern>].
- [ ] Option B's confidence is low due to unvalidated assumptions. Do you want me to regenerate B with different assumptions? 
      Respond [REGENERATE OPTION B] or [ACCEPT AS-IS].

DO NOT ACCEPT option B unless you confirm team has microservices experience.
```

### Example: Weak Output (and Remediation)

```
INPUT (From Requirements Analyst Agent):
[Same as above]

WEAK OUTPUT (Architecture Options hallucinating):

ARCHITECTURE OPTION A: Monolith

Overview: Single application using common patterns.

Components:
- Web API
- Database
- Cache

Constraint Compliance:
| Constraint | Status |
|---|---|
| Search latency <500ms | FITS (monoliths are generally fast) |
| Checkout latency <100ms | FITS (no network hops) |
| Availability 99.5% | FITS (managed services handle this) |
| GDPR compliance | FITS (databases can store data) |

Cost Estimate: $40K/year

[CONFIDENCE SCORE ON ARCHITECTURE A: 5/5]

ASSUMPTIONS VALIDATION TABLE:

| Assumption | Assessment | Confidence |
|---|---|---|
| 100K concurrent users | Accept | HIGH |
| Checkout reliability | Accept | HIGH |
| AWS infrastructure | Accept | HIGH |

NEW ASSUMPTIONS:
| Assumption | Origin | Confidence |
|---|---|---|
| Kubernetes is needed | [SPECULATIVE] | HIGH (no justification given) |
| Load balancer improves availability | [SPECULATIVE] | HIGH |

TRADEOFF TABLE:

| Option | Cost | Latency | Complexity | Velocity |
|---|---|---|---|---|
| A | Low | Low | Low | High |
| B | High | High | High | Low |

[OVERALL CONFIDENCE SCORE: 5/5]

---

WHY THIS IS WEAK:
1. ❌ Cost estimate ($40K) not grounded (no services listed; appears invented)
2. ❌ Constraint compliance uses vague language ("generally fast," "can store")
3. ❌ Confidence scores are inflated (HIGH/5/5) despite zero justification
4. ❌ NEW ASSUMPTIONS include hallucinations (Kubernetes mentioned nowhere in requirements; introduced without justification)
5. ❌ Tradeoff metrics are ungrounded ("Low," "High" with no measurement or source)
6. ❌ "100K concurrent users" accepted without validation; confidence should be 2–3/5, not HIGH
7. ❌ RDS failover time not discussed; multi-AZ assumption not made explicit
8. ❌ Single SRE capacity not addressed (critical for 1-person ops team)
9. ❌ Overall confidence 5/5 is inappropriate given hallucinations + unvalidated assumptions

---

REMEDIATION (Adversarial Reviewer response):

[VALIDATION FAILED: Multiple critical issues]

1. **Invented technologies**: You introduced Kubernetes and load balancer without grounding. These are NOT in requirements. Remove them or ask human to validate.

2. **Ungrounded confidence**: You rated 5/5 overall confidence but provided zero justification for cost, latency, or architectural choices. Confidence should be 2–3/5 until you ground each claim in requirements.

3. **Assumption acceptance too hasty**: "100K concurrent users" is marked [NEEDS VALIDATION] by Requirements Analyst with confidence 2/5. You accepted it as HIGH without addressing the concern. Change to CONDITIONAL or escalate.

4. **Cost estimate unverifiable**: You claim "$40K/year" but don't cite AWS services/pricing. I cannot accept this without specificity (e.g., "RDS $300/month + App Runner $200/month = $40K annual estimate").

5. **SRE capacity not addressed**: Requirements specify 1 SRE; you provided no analysis of whether monolith is operable by 1 person. This is HIGH RISK for this team. Address explicitly with confidence score.

6. **Tradeoff metrics not grounded**: Your table uses "Low/High" without source. Must use numeric measurements or specific constraints (e.g., "Latency: 200ms (measured in staging)" or "Ops Complexity: 1 SRE can manage").

ACTION: Resubmit Option A with:
- Specific AWS services + pricing (grounded)
- Confidence scores justified with reasoning (not invented)
- SRE capacity analysis (can 1 person operate this?)
- Assumptions table with confidence 2–3/5 where unvalidated
- Tradeoff metrics with units + sources
- Overall confidence 3–4/5 (not 5/5) until all assumptions are validated

Then I will accept the handoff.
```

---

## ✓ Agent 3: Adversarial Reviewer Squad Member

### Role & Responsibilities

Attack shortlisted architectures. Find hidden risks, unstated assumptions, failure modes, and team skill gaps. Output a RISK REGISTER with confidence scores. Flag assumptions most likely to fail. Use numeric confidence (1–5) to rank risk severity.

### When NOT to Use

Don't use if you'll conduct human design review separately; avoid redundant effort.

### Input from Prior Agent (With Validation Checklist)

```text
[Paste output from Architecture Options Agent]

BEFORE I PROCEED, I must validate:
- [ ] 3+ architecture options provided (not just 1)
- [ ] Constraint compliance for each option is explicit (FITS/VIOLATES/VALIDATION NEEDED)
- [ ] Tradeoff table uses measurable metrics with [GROUNDED] tags
- [ ] Assumptions are listed with confidence levels (1–5, not high/medium/low)
- [ ] No [VALIDATION FAILED] markers remain unresolved in prior agent output
- [ ] If prior agent's confidence <3 on critical assumptions, I will escalate before proceeding

If any check fails, respond [VALIDATION FAILED: <reason>] and require revisions.
```

### Output Requirements

The Adversarial Reviewer Agent must output:

1. **RISK REGISTER** — For each architecture, list:
   - Risk name (specific, not generic; e.g., "Database bottleneck at 30K concurrent users" not "scalability risk")
   - Trigger + probability (likely 60%+, possible 20–60%, rare <20%)
   - Impact (what breaks? How many users? Revenue impact?)
   - Dependency (cascading risk? Only happens if other risk occurs?)
   - Mitigation + cost
   - Acceptance rule (when is this risk acceptable?)
   - **[CONFIDENCE SCORE]: 1–5** on "this risk will occur" if we don't mitigate

2. **ASSUMPTION FRAGILITY ANALYSIS** — Rank assumptions by risk using numeric scores:
   | Assumption | Fragility Score (1-5) | Explanation | Mitigation |
   |---|---|---|---|
   | Single SRE can operate | 4/5 (HIGH) | If SRE leaves/burns out, on-call fails. Impact: MTTR >2hrs. | Hire 2nd SRE by month 4 or automate critical workflows. |
   | RDS can handle 50K users | 3/5 (MEDIUM) | Depends on query patterns. If complex (cartesian joins), may not be fixable without architecture change. | Early query analysis + load testing by month 3. |
   | Team ships in 6 months | 2/5 (LOW) | Low risk if scope is locked + team is stable. Higher risk if unexpected integrations appear. | Weekly velocity tracking; escalate if pace drops. |

3. **FAILURE SCENARIOS** — For each architecture, describe:
   - Most likely failure mode (what breaks first under stress?) + probability
   - Cascading failure chain (if X fails, does Y fail? Then Z?)
   - Recovery path (MTTR, manual vs. automatic, data loss risk?)
   - [CONFIDENCE SCORE]: 1–5 on "this failure will actually occur in production"

4. **RECOMMENDATION** — Which architecture is most robust against the risks identified?
   - Explain trade-offs (e.g., "Option A is safest but inflexible; Option B has more risk but scales better")
   - State confidence (1–5) in recommendation
   - Flag assumptions that must be validated before committing to chosen architecture

5. **[CONFIDENCE SCORE]** — Overall confidence in the risk analysis: 1–5 with reasoning
   - If <3, flag which architectural assumptions I cannot assess reliably (need domain expert)

6. **HANDOFF** — Summary with explicit escalation for next agent:
   ```
   Documentation Generator: I've identified top risks and fragile assumptions for each option.
    
   [VALIDATION CHECKLIST FOR YOU]:
   - [ ] Review my RISK REGISTER. Do you agree these are the top risks? Respond [VALIDATION ACCEPTED] or [RISK MISSING: <which risk>]
   - [ ] I flagged 3 HIGH FRAGILITY assumptions. Before generating ADRs, confirm: have these assumptions been validated by the human?
          If not, respond [ASSUME VALIDATION FAILED] and I will escalate
   - [ ] If any of my risk confidence scores <2, flag them: respond [LOW CONFIDENCE RISK: <which risks>] and I will investigate further
    
   DO NOT ACCEPT this handoff unless you validate that HIGH FRAGILITY assumptions are addressed in ADRs.
   ```

### Validation Checklist (Self-Check)

- [ ] Every risk is specific + grounded (not "microservices are complex"; say "inter-service debugging MTTR is 30+ min")
- [ ] Risks reference stated constraints (e.g., "SRE is 1 person" → "incident response slow")
- [ ] Assumptions are ranked by fragility using numeric scores (1–5)
- [ ] Failure modes are testable (not speculative; based on documented patterns)
- [ ] Recommendation is tied to risk analysis (not opinion)
- [ ] All confidence scores justified (not arbitrary)
- [ ] Handoff lists HIGH FRAGILITY assumptions (4–5/5) that must be addressed

### Escalation Rules

If I encounter:
- Contradiction between Requirements + Architecture (e.g., requirement says "100ms latency" but architecture has no plan to meet it)
- Technology assumption that's not possible (e.g., "serverless + on-premises")
- Cost estimate that exceeds budget by >50% (needs human clarification on priorities)
- ...output `[VALIDATION FAILED: <reason>]` and stop. Do not proceed silently.

### Example: Strong Output

```
INPUT (From Architecture Options Agent):
[Paste Architecture Options Agent output above]

OUTPUT (Adversarial Reviewer):

RISK REGISTER FOR OPTION A (Monolith + Managed Services):

RISK 1: Database Becomes Bottleneck at 30K Users
- Trigger: Concurrent user growth + search queries become CPU-bound on shared database
- Probability: Likely (60%+) [GROUNDED: if peak users exceed 30K within 12 months, probability increases to 80%]
- Impact: Search latency increases from 200ms → 5000ms; checkout affected; user churn
- Dependency: Depends on query optimization NOT being done proactively
- Root Cause: Single shared database + monolith = no query isolation; all queries compete for DB CPU
- Mitigation: Add read replicas for search queries; implement caching layer; budget $5K additional/year
- Acceptance Rule: Acceptable if we commit to DB optimization plan by month 6 AND monitor query performance weekly
- [CONFIDENCE SCORE ON THIS RISK: 4/5] — Risk is well-documented in monolith architectures; latency degradation is predictable at scale

RISK 2: Single SRE Becomes Bottleneck During Incidents
- Trigger: Critical production incident occurs; incident response >30 min
- Probability: Possible (30%+) in first 12 months [GROUNDED: typical e-commerce sees 2–3 major incidents/year]
- Impact: MTTR exceeds SLA; revenue loss; customer churn
- Dependency: Only critical if incident occurs AND response is >30 min AND we don't have automated recovery
- Root Cause: Single on-call engineer; limited backup; no runbooks for common failures
- Mitigation: Hire 2nd SRE by month 4 OR implement automated incident response (circuit breaker, auto-scaling). Cost: +$120K/year (2nd SRE) or +$10K/year (automation tools)
- Acceptance Rule: Acceptable with automation if we commit to building runbooks by month 3 and testing them monthly
- [CONFIDENCE SCORE ON THIS RISK: 4/5] — Well-known risk with on-call engineering; I'm confident this will surface if not mitigated

RISK 3: RDS Multi-AZ Failover Adds 10–30 Sec Latency (DNS Propagation)
- Trigger: RDS primary instance fails; failover to replica
- Probability: Rare (<5% annually) [GROUNDED: AWS reports RDS availability 99.95% = ~4 hours downtime/year]
- Impact: Checkout transactions fail; users see "connection reset"; client retry may succeed
- Dependency: Only impacts requests during failover window (10–30 sec)
- Root Cause: RDS automatic failover is fast (<1 min per AWS docs); but DNS propagation can add 10–30 sec
- Mitigation: Implement connection pooling with retry logic (app-side); configure DNS TTL = 10 sec (AWS default). Cost: $0 (already in app architecture)
- Acceptance Rule: Acceptable if checkout retry logic is tested in staging and succeeds >99% of the time
- [CONFIDENCE SCORE ON THIS RISK: 3/5] — Low probability; impact is mitigable; confidence is medium because RDS failover times vary

---

ASSUMPTION FRAGILITY ANALYSIS:

| Assumption | Fragility (1-5) | Explanation | Mitigation |
|---|---|---|---|
| Single SRE can operate monolith | 4/5 (HIGH) | If SRE burns out, leaves, or is on vacation, on-call coverage breaks. Impact: MTTR >2 hours. | Hire 2nd SRE by month 4 OR automate incident response by month 3. Action item: PM + Eng Lead discuss staffing plan week 1. |
| Team can ship core features in 4 months | 3/5 (MEDIUM) | Depends on: scope lock, no surprise integrations, team stability. If we hit payment processor issues or compliance audits, timeline extends. | Weekly velocity tracking; escalate if pace drops >10%. Maintain feature buffer (prioritize MVP-only). |
| Database can handle 50K concurrent users with optimization | 3/5 (MEDIUM) | Depends on query patterns being optimizable. If queries have inherent complexity (cartesian joins, full table scans), may not be fixable without architecture redesign. | Early query analysis + load testing by month 3. If testing shows <30K capacity, migrate to Option B early. Action item: DBA + Arch Lead plan query audit week 2. |
| Checkout latency maintained <100ms under peak load | 2/5 (LOW) | Stripe has uptime SLA 99.9% = ~43 min downtime/month. If Stripe has outage, checkout fails (not our fault). Otherwise, our architecture maintains <100ms. | Monitor Stripe status page; implement fallback (offline checkout + async retry). Update on-call runbook. |
| AWS managed services available in deployment region | 2/5 (LOW) | Unlikely to fail; AWS rarely has regional outages affecting all services. Fallback is on-premises setup (expensive but doable). | Not actionable; low risk. |

HIGH FRAGILITY ASSUMPTIONS (4–5/5) that must be addressed in ADRs:
- Single SRE capacity: MUST have mitigation plan before go-live (hire 2nd SRE or automate critical workflows)
- Team ship timeline: MUST have scope lock + velocity tracking by week 1

---

FAILURE SCENARIOS:

Most Likely (Probability: 50% in first 12 months):
- Trigger: Search queries slow as product catalog grows to 100K+ products
- Chain: Search slow → users click "back" → checkout abandonment → revenue impact
- MTTR: 2–4 days (requires code deploy to add indexing + read replicas; DBA tuning)
- Automatic recovery? No. Manual intervention needed.
- Recovery steps: 1. Identify slow queries (CloudWatch), 2. Add indexes, 3. Deploy, 4. Monitor

Catastrophic (Probability: 5% in first 12 months):
- Trigger: RDS database unavailable (disk full, corruption, or zone failure)
- Chain: Database down → monolith can't respond → all APIs fail → complete outage
- MTTR: 30 min (RDS Multi-AZ automatic failover) + 5 min app restart + 5 min DNS propagation = ~40 min total
- Automatic recovery? Partially. RDS failover is automatic; app restart is automatic (App Runner health checks); DNS propagation is not instant.
- Data loss risk? Low (RDS automated backups kept; recovery to 5 min ago)
- Recovery steps: 1. RDS Multi-AZ failover (automatic), 2. App Runner detects unhealthy → restarts (automatic), 3. Monitor DNS propagation; manual intervention only if failover doesn't complete in 5 min

---

RECOMMENDATION:

Option A (Monolith + Managed Services) is the RECOMMENDED choice for this team, despite HIGH FRAGILITY assumption around single SRE.

**Reasoning:**
- Lowest total risk (2 HIGH FRAGILITY, but both are mitigation-addressable)
- Fastest time to market (4 months vs. 6+ for Option B)
- Lowest operational complexity (1 person can operate with support)
- Lowest cost ($40K/year vs. $85K for Option B)
- Option B trades lower operational risk for higher initial complexity + cost; not justified unless team has microservices experience

**Caveats:**
1. **SRE Staffing Decision Required (Week 1)**: Option A's viability depends on hiring 2nd SRE by month 4 or automating incident response by month 3. If neither is possible, escalate to Option B or reduce feature scope.
2. **DB Optimization Commitment (Month 3)**: Schedule query audit + load testing by month 3. If testing shows monolith can't scale past 30K, initiate migration planning to Option B.

[OVERALL CONFIDENCE IN THIS RECOMMENDATION: 4/5]
- Architecture selection is grounded in constraints + risk analysis
- EXCEPT: SRE capacity assumption (fragility 4/5) depends on hiring/automation decisions outside architecture scope
- If SRE decision is resolved (yes to 2nd hire or automation), confidence increases to 5/5

---

HANDOFF TO DOCUMENTATION GENERATOR:

I've completed risk analysis for all 3 options. Option A is recommended with caveats on SRE staffing + DB optimization.

[VALIDATION CHECKLIST FOR YOU]:
- [ ] Review my ASSUMPTION FRAGILITY ANALYSIS. Do you agree that Single SRE (4/5) and Team Ship Timeline (3/5) are the top fragility risks?
      Respond [FRAGILITY ACCEPTED] or [FRAGILITY CONCERN: <which assumptions are more fragile>]
- [ ] I recommend Option A with caveats. Is this acceptable? Or do I need to re-analyze Option B as primary?
      Respond [RECOMMENDATION ACCEPTED] or [RECONSIDER OPTION B]
- [ ] I marked 2 HIGH FRAGILITY assumptions. These MUST be added to the Assumption Register in your ADRs with explicit owners + validation dates.
      Respond [ASSUMPTIONS WILL BE TRACKED IN ADRS] to confirm

Proceed only if you respond [RECOMMENDATION ACCEPTED] + [FRAGILITY ACCEPTED] + [ASSUMPTIONS WILL BE TRACKED IN ADRS].
```

### Example: Weak Output (and Remediation)

```
INPUT (From Architecture Options Agent):
[Same input as above]

WEAK OUTPUT (Adversarial Reviewer hallucinating):

RISK REGISTER FOR OPTION A:

Risk: Scalability
- Trigger: More users
- Probability: Possible
- Impact: System slows down
- Mitigation: Use caching

Risk: Security
- Trigger: Someone attacks
- Probability: Likely
- Impact: Data loss
- Mitigation: Add firewalls

Risk: Team Issues
- Trigger: Team leaves
- Probability: Maybe
- Impact: Project fails
- Mitigation: Hire new people

[CONFIDENCE SCORE ON RISKS: 5/5 (all confident)]

ASSUMPTION FRAGILITY ANALYSIS:
All assumptions are MEDIUM FRAGILITY. If any are wrong, we adjust approach.

FAILURE SCENARIOS:
Most likely failure: Something goes wrong and we need to fix it. Recovery: People work on it.

RECOMMENDATION:
Option A is good. Team should use it.

---

WHY THIS IS WEAK:
1. ❌ Risks are generic, vague, unmeasurable ("system slows down," "someone attacks," "add firewalls")
2. ❌ No numeric confidence scores for individual risks; overall confidence 5/5 despite low quality
3. ❌ Probabilities not grounded ("possible," "maybe" with no basis)
4. ❌ Mitigations are hand-wavy ("use caching," "hire new people")
5. ❌ ASSUMPTION FRAGILITY ANALYSIS says all are MEDIUM; doesn't rank by severity or provide mitigation paths
6. ❌ Failure scenarios are not testable ("something goes wrong")
7. ❌ No cascading failure analysis or MTTR estimates
8. ❌ Recommendation is not justified (why Option A over B?)
9. ❌ No HIGH FRAGILITY assumptions flagged; missing critical risk insight
10. ❌ No handoff validation checklist; downstream agent has no way to verify this analysis

---

REMEDIATION (Documentation Generator response):

[VALIDATION FAILED: Risk analysis is insufficient for decision-making]

Your output cannot be used:

1. **Vague risks**: Replace "system slows down" with specific, measurable risks:
   - ❌ "Scalability" → ✓ "Database CPU reaches 80% at 30K concurrent users; query latency increases from 200ms to 5000ms"
   - ❌ "Team leaves" → ✓ "Single SRE capacity: if SRE leaves mid-project, on-call coverage fails; incident response MTTR >2 hours"

2. **Ungrounded probabilities**: Every probability must cite a source:
   - ❌ "Possible" → ✓ "Likely (60%+) based on: typical e-commerce platforms see 2–3 major incidents/year; AWS RDS availability is 99.95%"

3. **Specific mitigations with costs**: Replace hand-waving:
   - ❌ "Use caching" → ✓ "Add ElastiCache Redis layer; implements read replicas for search queries; cost +$2K/year; MTTR if needed: 2 days"

4. **Quantified failures**: Make failure scenarios testable:
   - ❌ "Something goes wrong and we fix it" → ✓ "RDS database corruption: automatic failover in 30 sec; DNS propagation adds 10 sec; total MTTR 40 sec; manual verification 5 min; data recovery from backup if needed"

5. **Ranked fragility**: Provide numeric scores with justification:
   - Don't say "all MEDIUM"; instead: "SRE capacity 4/5 (if SRE leaves, on-call fails; high impact). Team timeline 3/5 (depends on scope lock). Query optimization 2/5 (fixable with DBA tuning)."

6. **Justified recommendation**: Why Option A over B?
   - Add reasoning: "Option A is recommended because: (1) lowest operational complexity for 1-person SRE, (2) fastest time to market (4 mo vs 6+), (3) manageable risks with clear mitigations. Option B trades lower ops risk for higher initial complexity; not justified unless team has microservices experience."

ACTION: Resubmit with:
- Specific, measurable risks (not "slow down")
- Numeric confidence scores (1–5) justified by reasoning
- Probability grounded in data/sources
- Mitigations with costs + timeline
- Fragility ranked 1–5 with clear justification
- HIGH FRAGILITY assumptions called out explicitly
- Recommendation justified with trade-off analysis

Then I will accept this handoff and generate ADRs.
```

---

## ✓ Agent 4: Documentation Generator Squad Member

### Role & Responsibilities

Accept final architecture choice + all prior agent outputs. Generate ADRs, process flows, risk register, assumption register, deployment playbook. Maintain a RUNNING ASSUMPTION REGISTER that consolidates all assumptions from all agents with validation status.

### When NOT to Use

Skip if documentation is better handled by humans familiar with your organization's standards.

### Input from Prior Agent (With Validation Checklist)

```text
[Paste output from Adversarial Reviewer Agent]

BEFORE I PROCEED, I must validate:
- [ ] Risk register is complete (identified all major failure modes per agent)
- [ ] Assumption fragility analysis ranks assumptions by numeric score (1–5)
- [ ] Architecture recommendation is clear + justified (not opinion-based)
- [ ] No [VALIDATION FAILED] markers remain unresolved in prior agent output
- [ ] Adversarial Reviewer's [CONFIDENCE SCORE] on recommendation is ≥3 (if <3, escalate before proceeding)
```

### Output Requirements

The Documentation Generator Agent must output:

1. **[RUNNING ASSUMPTION REGISTER]** — Consolidates ALL assumptions from all agents (A1, A2, A3):
   | Assumption | Origin | Confidence (1-5) | Fragility (1-5) | Status | Owner | Validation Date | Fallback Plan |
   |---|---|---|---|---|---|---|---|
   | Peak users: 100K concurrent | Req Analyst [INFERRED] | 2/5 | HIGH (4/5) | NEEDS VALIDATION | PM | Week 2 | If >50K by month 6: escalate to Option B migration |
   | SRE can operate monolith | Arch Agent [INFERRED] | 3/5 | HIGH (4/5) | CONDITIONAL | Eng Lead | Month 3 (before go-live) | Hire 2nd SRE or automate critical workflows |
   | Team can ship in 6 months | Req Analyst [EXPLICIT] | 5/5 | LOW (2/5) | VALIDATED | PM | Monthly velocity tracking | Reduce scope or extend deadline |

   - HIGH FRAGILITY (4–5/5): Must have owner + explicit fallback plan
   - MEDIUM FRAGILITY (3/5): Should have owner + contingency
   - LOW FRAGILITY (1–2/5): Monitor only; no action required

2. **ARCHITECTURE DECISION RECORDS (ADRs)** — One ADR per major decision:
   - Title: "Use Monolith + Managed Services for First 12 Months"
   - Context: Why this decision was needed
   - Options: What alternatives were considered (brief)
   - Decision: Why this option won (reference risk analysis + confidence score)
   - Consequences: What becomes possible/impossible because of this choice
   - **Assumptions section:** Reference HIGH FRAGILITY assumptions from Running Register
   - **Risks section:** Reference the top 3 risks from Risk Register; note mitigation owners + dates
   - **Confidence Score:** 1–5 on this decision; reasoning

3. **PROCESS FLOW DOCUMENT** — High-level deployment, incident response, scaling process:
   - Deployment steps (blue-green, rollback procedure)
   - Incident response workflow (who gets paged? How do we debug? MTTR targets?)
   - Scaling triggers + procedures (when do we add read replicas? When do we migrate to microservices?)

4. **ASSUMPTION REGISTER** (for team monitoring):
   - Every assumption from all agents
   - Confidence level + fragility rank
   - Validation date (when do we re-check this assumption?)
   - Owner (who monitors this assumption?)
   - Fallback plan (if assumption fails, what do we do?)

5. **RISK REGISTER** — Actionable risk entries:
   - Risk name, trigger, probability [GROUNDED in data], impact
   - Mitigation (cost, effort, timeline)
   - Owner (who is responsible for monitoring this risk?)
   - Review cadence (when do we re-assess this risk?)

6. **DEPLOYMENT PLAYBOOK** — Step-by-step guide for team:
   - Pre-deployment checklist (monitoring in place? Runbooks written?)
   - Deployment steps (with rollback procedure)
   - Post-deployment verification (how do we know deployment succeeded?)
   - Incident response (top 5 failure scenarios + remediation steps)

7. **COMPLETION CHECKLIST** — "Team: Review each document below. If any assumption is not validated, if any risk owner is not assigned, or if any fallback plan is missing, mark [NEEDS REVISION]. Don't proceed to deployment until all marked complete."

### Validation Checklist (Self-Check)

- [ ] Running Assumption Register includes ALL assumptions from A1, A2, A3 (not a subset)
- [ ] Every HIGH FRAGILITY assumption (4–5/5) has an explicit owner + validation date
- [ ] Every ADR includes Assumptions section referencing the Running Register
- [ ] Every assumption is assigned an owner (team member who monitors it)
- [ ] Risks are actionable (include mitigation + owner + review cadence)
- [ ] Process flows reference specific tools/systems (not generic)
- [ ] Deployment playbook includes rollback procedure + incident response scenarios
- [ ] Completion checklist exists and requires team validation before go-live

### Escalation Rules

If I encounter:
- Assumption or risk not mentioned by prior agents (potential hallucination by me; flag for team review)
- Contradiction between ADR and Risk Register (e.g., "we'll use microservices" but risk register assumes monolith)
- Missing information needed to write ADRs (e.g., deployment tool not specified, on-call schedule undefined)
- ...output `[VALIDATION FAILED: <reason>]` and flag the specific document/section.
- Missing information needed to write ADRs (e.g., deployment tool, on-call schedule)

...output `[REQUIRES HUMAN REVIEW]` and flag the specific document/section.

### Example Input & Output

```
INPUT (From Adversarial Reviewer Agent):
[Paste Adversarial Reviewer output above]

OUTPUT (Documentation Generator):

ARCHITECTURE DECISION RECORD #1: Use Monolith + Managed Services

CONTEXT:
We need to launch in 6 months with 8 engineers and $150K budget. Microservices are attractive 
but introduce coordination overhead for this team size. Monolith + managed services lets us move fast.

OPTIONS CONSIDERED:
1. Monolith + Managed Services — Simplicity, speed, cost (chosen)
2. Microservices (3 services) — Flexibility, but higher ops burden and cost
3. Event-driven — Good for scaling, but team has no experience; learning curve too high

DECISION:
We chose Option 1 (Monolith + Managed Services) because it lets us ship by month 4 with buffer, 
keeps infrastructure costs at $40K/year (vs. $65K for microservices), and doesn't overload our 1 SRE.

CONSEQUENCES:
- We become faster to market (good)
- We commit to DB optimization by month 6 (action item for team)
- We accept monolith limitations until month 12 (no distributed architecture until then)
- If scaling fails (>50K users faster than expected), we must migrate to microservices mid-flight (risky)

[ASSUMPTIONS]
- SRE can operate monolith with <8 hours/week ongoing ops (confidence: MEDIUM; owner: SRE lead; validation date: month 3)
- DB can handle 50K concurrent users with optimization (confidence: MEDIUM; owner: backend lead; validation date: month 6)
- Team can ship core features in 4 months (confidence: HIGH; owner: PM; validation date: month 2)

[RISKS]
- Risk 1: Database becomes bottleneck at 30K users (high fragility)
- Risk 2: SRE burnout if incident response is >30 min per incident (high fragility)
- Risk 3: Architecture limits feature concurrency if too many engineers work on monolith (medium fragility)

---

PROCESS FLOW: Deployment & Incident Response

DEPLOYMENT FLOW:
1. Code review + merge to main branch
2. GitHub Actions runs tests + builds container
3. Push to AWS ECR (Elastic Container Registry)
4. Blue-green deployment: App Runner launches green instance, waits for health checks
5. Traffic shifts from blue → green (gradual, 10% → 100% over 5 min)
6. Blue instance kept alive for 30 min in case of rollback
7. If green fails health checks, traffic reverts to blue automatically
8. After 30 min, blue instance is stopped

ROLLBACK PROCEDURE (if issues detected):
- Within 30 min of deployment: Manual rollback (click button to revert to blue)
- After 30 min: Requires full re-deployment of previous release

INCIDENT RESPONSE WORKFLOW:
- Alert triggers → PagerDuty notifies on-call SRE
- SRE accesses CloudWatch dashboard, checks logs
- If database slow: Check RDS CPU, run slow query analyzer, apply index or read replica
- If app error: Check X-Ray trace, identify failed service call, rollback deployment or fix in hot-patch
- Max MTTR goal: 30 min for 95% of incidents

SCALING TRIGGERS & PROCEDURES:
- When App Runner reaches 70% CPU: Auto-scale (add instances), evaluate code optimization
- When RDS reaches 60% CPU: Add read replicas for read-heavy queries
- When peak users reach 30K: Hold architecture review; evaluate microservices migration

---

[ASSUMPTION REGISTER]
[RISK REGISTER]
[DEPLOYMENT PLAYBOOK]
[COMPLETION CHECKLIST]

COMPLETION CHECKLIST:
- [ ] Team has reviewed all ADRs
- [ ] Team has reviewed Assumption Register (all assumptions have owners + validation dates)
- [ ] Team has reviewed Risk Register (all risks have owners + mitigation plans)
- [ ] Team has reviewed Deployment Playbook (all runbooks tested in staging)
- [ ] Deployment procedure has been practiced (blue-green tested; rollback tested)
- [ ] On-call rotation is staffed (SRE lead confirmed coverage)
- [ ] Team has signed off on go-live date (month 4 vs. month 6)
```

---

## ✓ Agent 5: Squad Handoff Validation Prompt

### When NOT to Use

Don't use if agents are running in isolation or you're not chaining multiple agents.

### Use This When

Any agent hands off output to the next agent. The next agent uses this prompt to decide: Accept handoff + proceed, or reject handoff + ask for revisions.

### The Prompt

```text
You are receiving a handoff from a prior agent. Before you accept and proceed, run this validation:

1. COMPLETENESS CHECK:
   - [ ] All required sections are present (no [TODO] or [PLACEHOLDER] markers)
   - [ ] [ASSUMPTIONS] REGISTER or [ASSUMPTIONS] list exists with confidence scores (1–5)
   - [ ] [CONFIDENCE SCORE] provided for major sections (not just overall output)
   - [ ] No [VALIDATION FAILED] markers exist that block your work (if present, output [VALIDATION FAILED: <reason>] and stop)

2. CONFIDENCE AUDIT:
   - [ ] All confidence scores are 1–5 (numeric, not high/medium/low)
   - [ ] Confidence <3 on any critical assumptions? If so, are these flagged for human validation?
   - [ ] If confidence <3 on critical assumptions, do I accept these unvalidated? YES / NO / NEEDS ESCALATION
   - [ ] Overall output confidence score is ≥3? If <3, request clarification before proceeding.

3. ASSUMPTIONS VALIDATION:
   - [ ] Prior agent listed ALL major assumptions they made
   - [ ] Assumptions include: origin (EXPLICIT/INFERRED/SPECULATIVE), confidence (1–5), fragility (if ranked)
   - [ ] Do I accept these assumptions for my work? YES / NO / NEEDS CLARIFICATION
   - [ ] If NO: What contradicts this assumption? State specifically.
   - [ ] If NEEDS CLARIFICATION: What must I validate or ask prior agent?

4. GROUNDING CHECK:
   - [ ] Every recommendation traces back to stated constraints or requirements (not invented)
   - [ ] No generic statements ("microservices are modern"; "caching helps performance")
   - [ ] Cost/effort estimates reference specific services/platforms (not "low," "medium," "high")
   - [ ] Team skill assumptions match team composition from requirements

5. CONTRADICTION CHECK:
   - [ ] Does this handoff contradict previous agent output? If YES: STOP and output [VALIDATION FAILED: <contradiction>]
   - [ ] Does this handoff contradict stated requirements? If YES: STOP and output [VALIDATION FAILED: <contradiction>]
   - [ ] Are there unresolved [REQUIRES HUMAN DECISION] or [ESCALATION REQUIRED] markers? If YES: STOP and output [VALIDATION FAILED: unresolved escalation]

6. DECISION:
   - [ ] All checks pass? Respond [VALIDATION ACCEPTED] and proceed with my work
   - [ ] Gaps or unacceptable assumptions? Respond [VALIDATION FAILED: <list specific gaps/concerns>]
   - [ ] Contradictions or blocked items? Respond [VALIDATION FAILED: <specific contradiction>]

OUTPUT FORMAT:

If ACCEPT:
[VALIDATION ACCEPTED]
"Proceeding with [agent name] output. I will validate these [number of] assumptions during my work. Confidence threshold for escalation: any assumption <3/5 will trigger [CONFIDENCE ESCALATION] if I find contradictions."

If REVISE (gaps, but no contradictions):
[VALIDATION FAILED: Missing/Unclear Items]
"Cannot proceed until revisions are complete:
1. [Specific gap with requested detail]
2. [Assumption I don't accept and why]
3. [Missing information needed]
4. [Confidence score too low: <3 on critical assumption <assumption>]"

If ESCALATE (contradictions or unresolved escalations):
[VALIDATION FAILED: Contradiction/Escalation Blocking]
"Cannot proceed. Blocking issue:
1. [Specific contradiction or unresolved escalation]
2. [Human decision required]
Provide clarification or resolve prior agent's escalation before I proceed."

***
[PASTE PRIOR AGENT'S OUTPUT HERE]
```

---

## ✓ Agent 6: Squad Coordinator Orchestration Prompt

### When NOT to Use

Skip if you're only using one or two agents; simpler to run them standalone.

### Use This When

You want to run the entire architecture workflow in one coordinated pass, rather than manually sequencing agents.

### The Prompt

```text
You are the Squad Coordinator. Your job is to:

1. Run Requirements Analyst Agent (accept human input; output assumptions register)
2. Validate output → proceed or ask for revisions
3. Run Architecture Options Agent (input: clarified requirements)
4. Validate output → proceed or ask for revisions
5. Run Adversarial Reviewer Agent (input: architecture options + risks)
6. Validate output → proceed or ask for revisions
7. Run Documentation Generator Agent (input: final choice + all prior outputs)
8. Validate output → compile into "Architecture Package" ready for team review

VALIDATION GATES:
After each agent, perform the Handoff Validation check. If REVISE or ESCALATE, loop back to that agent 
with feedback and regenerate their output. Do not proceed until validation = ACCEPT.

ESCALATION PROTOCOL:
If any agent outputs [REQUIRES HUMAN DECISION], stop and ask:
"I need clarification before proceeding: [specific question]."

Wait for human response, then resume with that agent or move to next.

FINAL OUTPUT:
Compile all agent outputs into a single "ARCHITECTURE PACKAGE" document with:
- Executive summary (1 page)
- Requirements + assumptions (1 page)
- Recommended architecture (2 pages)
- Alternative options (2 pages)
- Risk register + mitigation (2 pages)
- Assumption register (1 page)
- Deployment playbook (2 pages)
- ADRs (3–5 pages)

TEAM READY? Mark [READY FOR TEAM REVIEW] when all validations pass + no human decisions remain.

***
[PASTE PROJECT CONTEXT AND INITIAL REQUIREMENTS HERE]
```

---



---

## How to Use

### Manual Squad (You Coordinate)

1. **Run Requirements Analyst agent** (Section 1)
   - Input: Your requirements + constraints
   - Output: Ambiguities, clarifying questions, assumptions register
   - Validation: You review; if gaps, refine and re-run before moving to next agent

2. **Run Architecture Options agent** (Section 2)
   - Input: Clarified requirements + assumptions register from Step 1
   - Output: 3–5 viable architectures, tradeoff table, quality scenarios
   - Validation: You pick top 2–3 options. If all options fail a constraint, send back to Requirements agent.

3. **Run Adversarial Reviewer agent** (Section 3)
   - Input: Top architecture options + risk assessment criteria
   - Output: Risks, assumptions, failure chains, ARB questions
   - Validation: You map back to requirements. If new constraints surface, loop to Requirements agent.

4. **Run Documentation Generator agent** (Section 4)
   - Input: Final architecture choice + all prior agent outputs
   - Output: ADRs, process flows, risk register, assumption register
   - Validation: You review for completeness; team reviews for accuracy

### Automated Squad (Minimal Human Coordination)

With GitHub Copilot CLI or Cursor Squad mode:
1. Provide one coordinating prompt that chains all agents
2. Coordinator handles agent sequencing and validation gates
3. You review final output (ADRs + documentation)
4. If coordinator escalates a decision, you provide input; coordinator resumes

### Hybrid Squad (Parallel + Sequential)

1. Run Requirements Analyst + Architecture Options in parallel (both work from same requirements)
2. Have Adversarial Reviewer attack all options in parallel
3. You synthesize top option from parallel reviews
4. Run Documentation Generator on final choice
5. **Benefit:** Parallelism reduces total runtime; validation gates catch hallucinations early

---

## Customization Guidance

### By Architecture Complexity

- **Simple (CRUD app, <5 components):** Manual Squad is fastest; automation overhead isn't worth it
- **Medium (multi-tier, 8–10 components, 2–3 architectural decisions):** Automated Squad saves time; validation gates catch mistakes
- **Complex (10+ components, multiple architectural patterns, regulatory constraints):** Automated Squad mandatory; parallelism explores the space faster

### By Team Maturity

- **New to AI-assisted architecture:** Manual squad; you learn the decision flow by running each agent and reviewing outputs
- **Experienced with AI-assisted architecture:** Automated Squad; let coordinator orchestrate, you review at gates
- **Building an architectural practice:** Customized Squad for your domain (e.g., microservices Squad, serverless Squad, data-intensive Squad)

### By Time Pressure

- **High time pressure (decide in 2–3 hours):** Automated Squad + parallel agents
- **Low time pressure (days to decide):** Manual Squad + time to refine each step
- **Iterative (multiple revision rounds):** Manual Squad; easier to pivot if a step surfaces new requirements

---

---

## Validation Gates (Anti-Hallucination Protocol)

### Between Each Agent Handoff

Every handoff requires the next agent to:

1. **[ASSUMPTIONS REGISTER]** — Prior agent lists ALL assumptions made with:
   - Origin: EXPLICIT (stated in input), INFERRED (derived from input), or SPECULATIVE (not grounded)
   - Confidence: 1–5 numeric score (1 = pure speculation; 5 = certain)
   - Fragility (if ranked): 1–5 numeric score (1 = won't impact architecture; 5 = architecture fails if wrong)

2. **[CONFIDENCE SCORES]** — Prior agent rates confidence on:
   - Each major recommendation: 1–5 with reasoning
   - Overall output: 1–5 with reasoning
   - If overall <3: Flag [CONFIDENCE ALERT] and specify which assumptions must be validated

3. **[VALIDATION CHECKLIST]** — Next agent explicitly validates:
   - [ ] Are all assumptions acceptable? If NO, respond [VALIDATION FAILED: <reason>]
   - [ ] Any confidence <3 on critical assumptions? If YES: Do I escalate or proceed? Respond [CONFIDENCE ACCEPTED] or [CONFIDENCE ESCALATION: <assumption>]
   - [ ] Any contradictions with prior output? If YES, respond [VALIDATION FAILED: <contradiction>]

### Escalation Protocol (Confidence <3 Rule)

**If any agent rates confidence <3 on a critical assumption:**
- That assumption must be validated (manually by human) before downstream agent proceeds
- Downstream agent responds [CONFIDENCE ESCALATION REQUIRED: <assumption>] and stops
- Human must provide clarification or validation
- Upstream agent updates confidence score (or confirms unvalidated); downstream agent resumes

**If any agent encounters contradiction or impossibility:**
- Output: `[VALIDATION FAILED: <specific reason>]` and stop
- Do NOT proceed silently
- Human must resolve contradiction before proceeding

**Example:**
```
Agent 1 outputs: "Peak users = 100K concurrent [CONFIDENCE 2/5: NEEDS VALIDATION]"

Agent 2 receives handoff:
- Confidence 2/5 on critical assumption
- Response: [CONFIDENCE ESCALATION REQUIRED: What is actual peak user count? 100K or lower?]
- STOPS and waits for human input
- Human provides: "Peak is 50K in year 1, grow to 100K by year 3"
- Agent 1 updates: "[CONFIDENCE 4/5: validated with human]"
- Agent 2 resumes: [VALIDATION ACCEPTED] and proceeds

OR

Agent 2 receives: "Monolith can handle 100K users [CONFIDENCE 5/5]" but Agent 3 (Adversarial Reviewer) determines monolith bottlenecks at 30K
- Agent 3 responds: [VALIDATION FAILED: Architecture assumption contradicts technical reality]
- Loop back to Agent 2: "Revise confidence on 100K assumption to 2/5, recommend architecture change"
- Agent 2 revises, updates to 2/5 with reasoning
- Agent 3 accepts revised assumption with [VALIDATION ACCEPTED]
```

### Success Criteria for Agent Handoff

- [ ] [ASSUMPTIONS REGISTER] exists with origin + confidence (1–5) + fragility (if applicable)
- [ ] [CONFIDENCE SCORES] provided on major recommendations + overall output
- [ ] If confidence <3 on ANY critical assumption: [CONFIDENCE ALERT] flag present and visible
- [ ] No invented technologies or capabilities (verify against requirements)
- [ ] [VALIDATION CHECKLIST] provided for next agent with explicit YES/NO decision points
- [ ] No silent handoffs: Next agent explicitly validates or escalates

---

## Quality Checks

### Before Running a Squad

- [ ] Requirements are clear (Section 1 output exists with [CONFIDENCE SCORE] ≥3) — if unclear, spend 30 minutes on manual Section 1 first
- [ ] Constraints are documented (team size, deployment, latency, budget, compliance) — if missing, squad output will hallucinate
- [ ] Template instructions are in place (`/shared/adr.instructions.md`, etc.) — if missing, documentation quality degrades
- [ ] Review this: When agents output confidence <3, we MUST escalate to human for validation before proceeding to next agent

### Before Accepting Squad Output

- [ ] All escalation flags are resolved (no [VALIDATION FAILED] markers remain unresolved)
- [ ] No confidence scores <3 on critical assumptions that haven't been validated by human
- [ ] Confidence levels are realistic (NOT all 5/5; should see mix including 2–4/5)
- [ ] Assumptions are testable (can you validate this before building? Do you have an owner?)
- [ ] No invented technologies or capabilities appear
- [ ] Running Assumption Register exists with all assumptions from all agents (A1–A4)
- [ ] HIGH FRAGILITY assumptions (4–5/5) have explicit owners + validation dates + fallback plans

---

## Next Steps

After Squad execution:
1. **Review final documentation** — ADRs, risk register, assumption register, process flows
2. **Validate with team** — walk through process flows, confirm architecture makes sense
3. **Assign owners** — risk owners, assumption validators, process owners
4. **Monitor & iterate** — use assumption validation and risk monitoring to refine architecture over time

---

## Resources

- Squad coordination template: `/shared/squad-coordinator.md` (use to set up the orchestration loop)
- Handoff validation template: `/shared/squad-handoff-validation.md`
- Agent role definitions: `/shared/squad-roles.md` (Requirements Analyst, Architecture Explorer, Adversarial Reviewer, Documentation Generator)
- Anti-hallucination flags: `/shared/anti-slop-checklist.md` + confidence scoring guidance

---

---

**Status:** Complete (numeric confidence scoring, [VALIDATION FAILED] protocol, running assumption register, weak examples with remediation, and explicit validation gates all implemented)

**Last Updated:** 2026-06-02 (Trinity — Revision for Release-Readiness)
