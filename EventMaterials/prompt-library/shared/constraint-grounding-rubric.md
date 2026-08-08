# Constraint Grounding Rubric

Evaluation framework for verifying that architecture recommendations actually fit stated constraints (not generic best practices).

---

## Purpose

AI often generates plausible but hollow recommendations. This rubric checks whether each architecture choice is actually grounded in your stated constraints or is just standard advice ("microservices scale" "use caching" "add redundancy").

---

## Grounding Check Matrix

### Technology Choice Grounding

**For every technology recommendation, verify it answers:**

| Question | Weak Answer | Strong Answer |
|----------|-------------|----------------|
| **Why this tech?** | "It's industry standard" | "Team has 5 engineers with Node.js expertise (constraint: 6-month TTM); learning Go adds 2 months (violates TTM)" |
| **What problem does it solve?** | "Better performance" | "Solves 10K req/sec throughput target; alternative would require caching layer + adds operational complexity" |
| **What constraints does it fit?** | [Not referenced] | Fits: team skill (5/5), timeline (6 months), budget ($50K: RDS managed service cheaper than self-managed); violates: none |
| **What constraints does it violate?** | [Not mentioned] | Violates: multi-region requirement (RDS single-region; mitigation: use Aurora with cross-region replica, +$2K/yr) |
| **What are the tradeoffs?** | "More scalable" | "PostgreSQL: Scales to 100K writes/sec with indexing; alternative NoSQL: scales to 1M writes/sec but violates consistency requirement" |
| **What could go wrong?** | [Not addressed] | Bottleneck at 100K concurrent users (mitigation: sharding by customer_id; 2-week rewrite if needed) |

---

### Constraint Compliance Scoring

**Rate each recommendation on constraint fit:**

| Constraint | Fits? | Evidence | Mitigation (if needed) |
|-----------|-------|----------|----------------------|
| Team size (3 people) | ✓ FITS | Monolith easier for small team | N/A |
| Team skill (no K8s exp.) | ✓ FITS | Use App Runner (managed); no ops expertise needed | N/A |
| Timeline (6 months) | ✓ FITS | No new tech learning required | N/A |
| Budget ($50K/year) | ✓ FITS | App Runner: $2K/month = $24K/year; within budget | N/A |
| Latency <500ms | ⚠ NEEDS VALIDATION | Untested with real load; might need caching | Load test in month 1; if p99 > 500ms, add Redis |
| Availability 99.5% | ✓ FITS | RDS Multi-AZ automatic failover; SLA 99.95% | N/A |
| GDPR (EU only) | ✓ FITS | RDS in eu-west-1 region; no cross-border transfer | N/A |
| Scaling to 100K users | ✗ VIOLATES | Monolith bottleneck at 10K concurrent; rewrite needed | Plan microservices rewrite for year 2; document migration path now |

**Scoring:** FITS (green) | NEEDS_VALIDATION (yellow) | VIOLATES (red)

**Rule:** If any RED items exist, either (a) change architecture, (b) challenge constraint, or (c) mark [REQUIRES_VALIDATION] + plan mitigation.

---

## Grounding Depth Assessment

### Depth Level 1: Generic (❌ Weak)

**Example:** "Use microservices for scalability"

**Problems:**
- No reference to constraints
- Could apply to any project
- No specificity about YOUR situation

**Reframe to Depth 5:**
"Use microservices to enable 5 independent teams to deploy features without coordination (constraint: feature velocity = 2–3 features/sprint). Monolith would require coordinated releases, slowing velocity to 1 feature/sprint."

---

### Depth Level 2: Aspirational (❌ Weak)

**Example:** "Kubernetes enables auto-scaling and high availability"

**Problems:**
- Describes Kubernetes capabilities, not YOUR use case
- True but not specific to constraints
- Hides learning curve / ops burden

**Reframe to Depth 5:**
"Kubernetes is rejected. Rationale: Team has zero K8s experience. Learning curve = 3 months (consumes half of 6-month timeline). Operational burden = 1 FTE SRE minimum. We don't have ops budget. Alternative: AWS App Runner (managed Kubernetes) fits constraints: requires 0 Kubernetes expertise, $2K/month cost."

---

### Depth Level 3: Contextual (✓ Better)

**Example:** "Use PostgreSQL because team has SQL expertise"

**Problems:**
- Mentions team constraint
- But misses other constraints
- Doesn't show tradeoff analysis

**Reframe to Depth 5:**
"Use PostgreSQL because: (1) Team has 4 engineers with 5+ years SQL experience (TTM constraint: no learning curve). (2) ACID transactions required (consistency requirement). (3) Scales to 100K writes/sec (satisfies scale projection). Alternative: DynamoDB would scale to 1M writes/sec but violates consistency requirement + requires learning (team has no NoSQL experience). Decision: PostgreSQL."

---

### Depth Level 4: Grounded (✓ Good)

**Example:** "Monolith preferred for 3-person team with 6-month deadline. Scale to 10K users. If scale exceeds 10K, rewrite to microservices (2–3 weeks, planned in year 2)."

**Strengths:**
- References constraints (team size, timeline, scale)
- Shows decision logic
- Documents plan if constraint changes

**Still missing:**
- Proof constraint will actually be a bottleneck
- Specific failure modes if bottleneck occurs

---

### Depth Level 5: Fully Grounded (✓ Excellent)

**Example:**
"DECISION: Monolith + PostgreSQL

CONSTRAINTS SATISFIED:
✓ Team size (3): Monolith simplest architecture for coordination; microservices would require async patterns (new to team)
✓ Timeline (6 months): No new technology learning; team immediately productive
✓ Budget ($50K/year): Single instance RDS + App Runner = $24K/year; under budget
✓ Latency <500ms: Single instance responds in <100ms; Redis cache available if needed
✓ Availability 99.5%: RDS Multi-AZ automatic failover; SLA meets target
✓ Scale (1K → 10K): Monolith handles 10K concurrent; load tested successfully

CONSTRAINTS WITH CAVEATS:
⚠ Scale beyond 10K: Monolith bottleneck at database write throughput (~100K writes/sec); mitigation: Migrate to event-driven + microservices (estimated 2–3 week rewrite); plan started now

RATIONALE:
- Minimizes complexity for small team (maximize TTM)
- Proven technology (PostgreSQL mature, team skilled)
- Clear growth path (document microservices strategy now; execute in year 2 if needed)

FALLBACK:
- If scale reaches 10K faster than projected: Trigger microservices rewrite decision (ADR-005)
- If team needs faster feature velocity: Add feature flags + branching strategy (doesn't require rewrite)

VALIDATION:
- Load test in month 1: Confirm <500ms p99 latency at projected 1K users
- Monitor scaling metrics monthly: Alert if trending toward 5K+ users (gives 2–3 month lead time for rewrite)
"

**Why Depth 5 is excellent:**
- Every constraint explicitly checked
- Tradeoffs documented
- Fallback plan exists
- Validation milestones clear
- Reusable for future decisions

---

## Grounding Rubric Checklist

**For each architecture recommendation, verify:**

### Requirement 1: Constraint Inventory Present
- [ ] All stated constraints listed (team, budget, timeline, performance, compliance, scale, etc.)
- [ ] No invented constraints added
- [ ] Missing constraints identified and flagged [NEEDS_CLARIFICATION]

### Requirement 2: Constraint Fit Analysis
- [ ] For each constraint: Explicitly marked FITS / VIOLATES / NEEDS_VALIDATION
- [ ] For FITS: Evidence provided (e.g., "RDS SLA 99.95% exceeds 99.5% requirement by 2x margin")
- [ ] For VIOLATES: Stated clearly; not hidden (e.g., "Kubernetes learning curve 3 months violates 6-month TTM if started in month 1")
- [ ] For NEEDS_VALIDATION: Plan to validate exists (e.g., "Load test latency in month 1")

### Requirement 3: Tradeoff Analysis
- [ ] Alternative options listed (minimum 2)
- [ ] Each alternative checked against constraints
- [ ] Chosen option justified by constraint fit (not generic "best practice")
- [ ] If no option fits all constraints: Priority documented (which constraints override others?)

### Requirement 4: Failure Mode Documentation
- [ ] Scaling limits identified (what breaks first if load 10x?)
- [ ] Cost scaling model (does cost scale linearly with growth?)
- [ ] Skill gap risks (what if team can't learn this?)
- [ ] Dependency risks (what if third-party service fails?)

### Requirement 5: Fallback Plan
- [ ] If constraint changes: What architecture change is needed?
- [ ] Timeline & effort: How long to rewrite if needed?
- [ ] Cost of fallback: Is it acceptable?
- [ ] Example: "If scale exceeds 10K users: Migrate to microservices (2-week effort; $500 additional infrastructure cost)"

### Requirement 6: Validation Milestones
- [ ] Clear checkpoints to reassess constraints
- [ ] Success criteria for each checkpoint (specific metric, not vague)
- [ ] Trigger condition to escalate if validation fails
- [ ] Example: "Month 1 checkpoint: Load test confirms <500ms p99 latency. If latency >800ms, escalate for caching strategy."

### Requirement 7: Assumption Inventory
- [ ] Every assumption documented (team skill, vendor SLA, market size, etc.)
- [ ] Assumptions marked with confidence (1–5)
- [ ] High-risk assumptions flagged [HIGH_PRIORITY_VALIDATION]
- [ ] Example: "ASSUMPTION: Team can learn Kubernetes in 3 months (confidence 2/5; unvalidated; high risk)"

### Requirement 8: No Generic Filler
- [ ] Zero instances of "better," "faster," "simpler" without metrics
- [ ] Zero references to "best practices" without grounding
- [ ] Zero invented capabilities (e.g., "automatic" without mechanism)
- [ ] All claims reference specific constraints or evidence

---

## Grounding Score Calculation

**For a recommendation, calculate grounding score:**

| Requirement | Met? | Weight | Score |
|-------------|------|--------|-------|
| Constraint inventory present | ✓ | 15% | 15 |
| Constraint fit analysis complete | ✓ | 25% | 25 |
| Tradeoff analysis thorough | ✓ | 20% | 20 |
| Failure mode documented | ✗ | 15% | 0 |
| Fallback plan present | ✓ | 10% | 10 |
| Validation milestones clear | ✓ | 10% | 10 |
| Assumption inventory complete | ✓ | 5% | 5 |

**Total Grounding Score:** 85/100 (Good; missing failure mode analysis)

**Guidance:**
- **90–100:** Fully grounded; ready for approval
- **75–89:** Good grounding; address missing items before approval
- **60–74:** Partial grounding; requires significant revision
- **<60:** Not grounded; restart analysis with constraint-driven reasoning

---

## Using Rubric in Code Review

**When reviewing architecture doc, ask:**

1. "List all constraints mentioned. Are any missing?" (Requirement 1)
2. "For each constraint, is fit/violation explicit?" (Requirement 2)
3. "What alternatives were considered? Why wasn't each chosen?" (Requirement 3)
4. "What's the failure mode? What breaks when load 10x?" (Requirement 4)
5. "If constraint changes, what's plan B?" (Requirement 5)
6. "What metrics prove this will work?" (Requirement 6)
7. "List assumptions. Which are risky?" (Requirement 7)
8. "Does this sound generic or specific to our project?" (Requirement 8)

**Red flags (incomplete grounding):**
- ❌ Recommendation doesn't mention team size (fundamental constraint)
- ❌ "This scales" without saying to what scale
- ❌ "Modern best practice" without constraint tie-in
- ❌ No mention of cost / budget fit
- ❌ No fallback plan documented

---

## Grounding Template for Architecture Recommendations

```markdown
## Recommendation: [Technology or Approach Name]

### Constraints Satisfied
✓ [Constraint 1]: [Evidence it fits]
✓ [Constraint 2]: [Evidence it fits]
⚠ [Constraint 3]: [Caveat; mitigation]
✗ [Constraint 4]: [Violates; explain why worth it or propose alternative]

### Rationale
[Explain why this choice best fits constraints; not just "industry standard" or "popular"]

### Tradeoff Analysis
| Alternative | Fit | Why Not Chosen |
|---|---|---|
| [Option A] | [Fits some constraints; violates others] | [Why Option 1 is better] |
| [Option B] | [Fits some constraints; violates others] | [Why Option 1 is better] |

### Failure Modes
[What breaks if load 10x? If team scales? If constraint changes?]

### Fallback Plan
[If this doesn't work, what's plan B? Timeline and cost?]

### Validation Plan
- Milestone 1: [What to test + success criteria] by [date]
- Milestone 2: [Next test] by [date]

### Assumption Registry
| Assumption | Confidence | Risk |
|---|---|---|
| [Team can learn X] | [1–5] | [If false, what breaks?] |

### Grounding Score: __/100
[Self-rate on rubric above; identify gaps]
```

Use this template to ensure every recommendation is fully grounded.
