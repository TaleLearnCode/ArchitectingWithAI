# Squad Handoff Validation Template

Protocol for validating handoffs between squad agents. Use this to ensure each agent accepts the previous agent's output and flags gaps before proceeding.

---

## Purpose

Handoff validation prevents hallucinations, assumptions, and errors from cascading through the multi-agent pipeline. Each agent validates the previous agent's output before building on it, or responds with `[VALIDATION FAILED]` to trigger a loop back.

---

## Validation Protocol

### Step 1: Receive & Audit Previous Agent's Output

When Agent N+1 receives output from Agent N:

```
RECEIVE CHECKLIST:
□ All required sections present (e.g., Requirements + Assumptions Register + confidence scores)
□ No [VALIDATION FAILED] or [INPUT MISSING] markers unresolved
□ No unaddressed [CLARIFICATION NEEDED] flags
□ Confidence scores justified (1–5 with reasoning)
□ All assumptions marked with origin + confidence
□ Critical constraints explicitly checked against requirements
```

### Step 2: Validate Against Known Requirements

```
REQUIREMENT VALIDATION:
□ Does output respect all stated constraints? (team size, budget, latency, timeline, compliance)
□ Are any constraints VIOLATED? (Explicitly marked, not hidden)
□ Are any constraints UNTESTED? (Marked [NEEDS VALIDATION])
□ Are numeric targets consistent with stated requirements?
```

### Step 3: Validate Assumptions

```
ASSUMPTIONS VALIDATION:
□ Every assumption from prior agent has confidence score (1–5)
□ Confidence <3: marked for human validation (not hidden)
□ New assumptions created in prior phase marked with origin (EXPLICIT/INFERRED/SPECULATIVE)
□ No invented assumptions (claims not grounded in stated inputs)
□ Assumption origins traced (can I verify where this came from?)
```

### Step 4: Validate Confidence Scoring

```
CONFIDENCE VALIDATION:
□ Confidence scores have explicit justification (why 3/5, not 4/5?)
□ Scores are realistic (not all 5/5; should see distribution 2–5)
□ Scores <3 on critical items flagged + escalated
□ Scores >4 justified with evidence (not just assumed correct)
□ Confidence correlates with grounding (speculative claims get lower scores)
```

### Step 5: Decision Point

```
IF all validation checks pass:
  → Respond: [VALIDATION ACCEPTED]
  → Proceed to next phase

IF any validation check fails:
  → Identify specific failure: [VALIDATION FAILED: <category>, <reason>]
  → Do NOT proceed to next phase
  → Escalate to prior agent for revision OR escalate to human
```

---

## Validation Decision Tree

```
Does output have [INPUT MISSING] or unresolved [VALIDATION FAILED] markers?
├─ YES → Respond [VALIDATION FAILED: unresolved input/validation gaps]
│        Escalate to prior agent + human
└─ NO → Continue to next check

Do all stated constraints get explicit FITS/VIOLATES/NEEDS VALIDATION check?
├─ NO → Respond [VALIDATION FAILED: constraint untested]
│       Request prior agent to complete constraint audit
└─ YES → Continue to next check

Are all confidence scores justified with reasoning?
├─ NO → Respond [VALIDATION FAILED: unjustified confidence scores]
│       Request prior agent to add reasoning
└─ YES → Continue to next check

Are assumptions <3/5 flagged for human validation?
├─ NO → Respond [VALIDATION FAILED: unvalidated assumptions]
│       Request prior agent to flag confidence <3
└─ YES → Continue to next check

Does output claim FITS on all constraints but reasoning is weak?
├─ YES → Respond [VALIDATION FAILED: overconfident constraint mapping]
│        Request prior agent to justify or mark as NEEDS VALIDATION
└─ NO → Proceed with [VALIDATION ACCEPTED]
```

---

## Validation Checklist by Phase

### Phase 1 → Phase 2: Analyst Handoff

**Requirements Analyst output to Architecture Explorer**

```
VALIDATION CHECKLIST (Explorer verifies):

Input Completeness:
□ [INPUT MISSING] fail-fast validation completed (all required fields present)
□ Functional requirements numbered and traced to explicit stakeholder input
□ Non-functional requirements have numeric targets (not "fast," "scalable")
□ Constraints inventory complete (team size, budget, latency, deployment, timeline, compliance)

Requirements Quality:
□ Functional requirements are testable (not architectural)
□ Non-functional requirements have units and thresholds (e.g., "<500ms p99", not "<fast p99")
□ Requirements do NOT invent capabilities not mentioned by stakeholder
□ Ambiguities explicitly listed as questions for stakeholder

Assumptions Quality:
□ Every assumption marked with origin: EXPLICIT / INFERRED / SPECULATIVE
□ Every assumption has confidence score (1–5) with reasoning
□ Confidence <3: flagged [NEEDS VALIDATION] for Explorer to escalate to human
□ Assumptions register includes: dependency (what breaks if false), validation plan

Overall Confidence:
□ Phase 1 [CONFIDENCE SCORE] provided (1–5) with reasoning
□ Score <3: Explorer must validate with human before proceeding

Decision:
If all checks pass → [VALIDATION ACCEPTED]
If ANY check fails → [VALIDATION FAILED: <category>] + details for revision
```

---

### Phase 2 → Phase 3: Explorer Handoff

**Architecture Explorer output to Adversarial Reviewer**

```
VALIDATION CHECKLIST (Reviewer verifies):

Architecture Options Quality:
□ 3+ distinct options provided (not variations on one theme)
□ Each option has component breakdown (not abstract)
□ Each option has explicit constraint compliance check (FITS/VIOLATES/NEEDS VALIDATION per constraint)
□ Cost estimates reference specific services (not "low"/"medium"/"high")
□ Team fit analysis specific (how many engineers, required skills, learning curve)

Assumptions Validation:
□ Phase 1 assumptions re-validated in Phase 2 (table: ACCEPTED/FAILED/NEEDS CLARIFICATION)
□ New assumptions created during option generation marked with origin + confidence
□ Assumptions <3/5 flagged [CONFIDENCE ALERT] for human escalation
□ No assumptions marked FAILED without explanation of why they fail

Tradeoff Analysis:
□ Tradeoff table includes: cost, latency, ops complexity, team velocity, risk level
□ All metrics grounded: [GROUNDED: <source>] tags present (not opinion)
□ All options compared on same criteria (not cherry-picked metrics)
□ Recommendation clear (which option + why)

Overall Confidence:
□ Phase 2 [CONFIDENCE SCORE] provided (1–5) with reasoning
□ Score <3: Reviewer must challenge reasoning before accepting
□ Score justification includes which assumptions are unvalidated

Decision:
If all checks pass → [VALIDATION ACCEPTED]
If ANY check fails → [VALIDATION FAILED: <category>] + details for revision
```

---

### Phase 3 → Phase 4: Reviewer Handoff

**Adversarial Reviewer output to Documentation Generator**

```
VALIDATION CHECKLIST (Documentation Generator verifies):

Risk Analysis Quality:
□ Each architecture has ≥3 failure mode scenarios documented
□ Risks are specific (not "complexity may be high")
□ Each risk has: likelihood, impact, mitigation, owner, review frequency
□ Hidden risks identified (team skill gaps, vendor lock-in, scaling cliffs, regulatory)

Challenge Analysis Quality:
□ Assumptions from Phase 1–2 are questioned with reasoning
□ Trade-off claims verified (e.g., "you said low ops complexity; here's why that's wrong")
□ Recommendations include contingency plans for <3-confidence assumptions

Overall Confidence:
□ Phase 3 [CONFIDENCE SCORE] provided (1–5) with reasoning
□ All [HIGH_RISK] items flagged for CTO approval
□ Recommendation clear (final architecture choice + rationale)

Consolidated Assumptions Register:
□ All assumptions from Phase 1–3 compiled in single register (A1–A4)
□ Each includes: origin, confidence, validation plan, deadline, owner
□ Consolidated confidence score calculated (weighted average? minimum? explicit rule?)

Decision:
If all checks pass → [VALIDATION ACCEPTED]
If ANY check fails → [VALIDATION FAILED: <category>] + details for revision
```

---

## Response Templates

### When Validation Passes

```
[VALIDATION ACCEPTED]

Your output meets all acceptance criteria:
✓ Constraints checked: FITS/VIOLATES/NEEDS VALIDATION per option
✓ Assumptions validated: <N> from prior phase ACCEPTED, <M> new assumptions created
✓ Confidence scores justified: range 2–5, no unrealistic 5/5 claims
✓ Tradeoff analysis grounded: all metrics [GROUNDED: <source>]

Proceeding to [next phase].
```

---

### When Validation Fails

```
[VALIDATION FAILED: <category>]

Issues preventing advancement to next phase:

1. [Specific issue #1]
   - Evidence: [what I found]
   - Request: [what needs to be fixed]

2. [Specific issue #2]
   - Evidence: [what I found]
   - Request: [what needs to be fixed]

Please revise and resubmit. I will re-validate before proceeding.
```

---

### When Escalation Needed

```
[CONFIDENCE ESCALATION REQUIRED]

The following items require human decision before I can proceed:

1. [Assumption / Risk / Trade-off]
   - Current confidence: <score>/5
   - Issue: [why I cannot proceed at this confidence]
   - Required: [CTO / stakeholder / security lead] approval

Once human validates, I will proceed to next phase.
```

---

## Escalation Triggers

| Trigger | Response | Action |
|---------|----------|--------|
| Confidence <3/5 on critical assumption | [CONFIDENCE ESCALATION] | Wait for human validation before proceeding |
| Assumption marked FAILED but no explanation | [VALIDATION FAILED: unjustified assumption failure] | Request prior agent to explain why it failed |
| Constraint VIOLATED but no mitigation | [VALIDATION FAILED: unmitigated constraint violation] | Request prior agent to explain risk acceptance |
| Cost estimate referenced non-specific services | [VALIDATION FAILED: non-grounded cost] | Request prior agent to cite actual AWS/vendor pricing |
| Confidence >4/5 but <3 supporting evidence | [VALIDATION FAILED: overconfident] | Request prior agent to justify high confidence |
| [INPUT MISSING] or [VALIDATION FAILED] from prior phase unresolved | [VALIDATION FAILED: prior phase not closed] | Escalate to prior agent + human for resolution |

---

## Validation Failure Loop

**When Agent N+1 responds [VALIDATION FAILED]:**

1. Agent N receives [VALIDATION FAILED] + specific reasons
2. Agent N revises output:
   - Adds missing information
   - Addresses failed assumptions
   - Justifies confidence scores
   - Escalates human decisions if needed
3. Agent N resubmits revised output
4. Agent N+1 re-validates using same checklist
5. Loop continues until [VALIDATION ACCEPTED] or human intervention

**Max loop iterations:** 3 times. If not resolved after 3 iterations, escalate to human.

---

## Consolidated Assumptions Register (Phase 4 Gate)

**Before finalizing architecture, Documentation Generator consolidates all assumptions:**

```
CONSOLIDATED ASSUMPTIONS REGISTER

| ID | Assumption | Origin | Phase Created | Confidence | Validation Plan | Deadline | Owner | Status |
|----|-----------|--------|---|---|---|---|---|---|
| A1-001 | Team can learn Kubernetes in 3 months | INFERRED | 1 | 2/5 | Month 1 checkpoint: PoC deployment | Month 1, week 2 | Platform Lead | VALIDATING |
| A1-002 | Payment Gateway latency <200ms p99 | VENDOR CLAIM | 1 | 3/5 | Load test: 1K concurrent req, 24h measurement | Month 1, week 3 | SRE Lead | NOT STARTED |
| A2-001 | Single SRE can operate monolith | INFERRED | 2 | 3/5 | Month 2: simulate incident response (<30 min) | Month 2, week 1 | SRE Lead | NOT STARTED |
| A3-001 | Ops burden <0.5 FTE once stabilized | SPECULATIVE | 3 | 2/5 | Production run 30 days; measure incident load | Month 4, week 2 | On-call Lead | NOT STARTED |

VALIDATION GATES:
□ All assumptions with confidence <3: have explicit owner + deadline
□ All HIGH_RISK assumptions (4–5/5 confidence): have fallback plan
□ All validation deadlines: before critical path decision (e.g., before architecture lock-in)
□ Escalation contacts: assigned for each assumption
```

---

## Quality Checks

**Before accepting ANY handoff:**

- [ ] **Completeness:** All required sections present (not "TBD")
- [ ] **Clarity:** Reasoning is explicit (not "obviously correct")
- [ ] **Grounding:** Claims reference specific evidence (not opinions)
- [ ] **Confidence:** Scores justified with reasoning (not arbitrary)
- [ ] **Escalation:** All <3-confidence items flagged (not hidden)
- [ ] **Traceability:** Origins documented (can I audit the reasoning chain?)
- [ ] **No Hallucination:** No invented requirements or capabilities
- [ ] **Constraint Audit:** All stated constraints explicitly checked
- [ ] **Assumption Tracking:** All assumptions marked with origin + confidence
- [ ] **Fallback Plans:** High-risk assumptions have contingencies documented

---

## Anti-Patterns to Reject

| Anti-Pattern | Response |
|---|---|
| "Confidence 5/5, but assumption is speculative (not tested)" | [VALIDATION FAILED: unjustified confidence] |
| "All options marked FITS, but trade-off analysis shows one dominates" | [VALIDATION FAILED: incomplete trade-off analysis] |
| "Cost estimate: 'low'/'medium'/'high'" | [VALIDATION FAILED: non-grounded cost] |
| "[INPUT MISSING] marked, but handoff proceeds anyway" | [VALIDATION FAILED: unresolved input gap] |
| "Confidence scores all 4–5; no 2–3 range" | [VALIDATION FAILED: unrealistic confidence distribution] |

---

## Sign-Off Template

**When Phase 4 (Documentation) is complete and validated:**

```
ARCHITECTURE PACKAGE READY FOR APPROVAL

Summary:
- Decision: [Selected architecture option]
- Rationale: [Why chosen over alternatives]
- Confidence: [Overall confidence 1–5]
- Risk level: [Low / Medium / High]

Assumptions Consolidated:
- Total assumptions: [N]
- Confidence ≥3: [M] (approved to proceed)
- Confidence <3: [P] (flagged [NEEDS VALIDATION], validation deadlines assigned)

Critical Path Blockers:
- [If any high-risk assumptions block implementation, list here]

Ready for:
☐ Lead Architect approval
☐ Stakeholder review
☐ Team implementation
☐ [Other gates]

Sign-off by: [Role] Date: [YYYY-MM-DD]
```

---

**Last Updated:** 2026-06-02
