# Squad Coordinator Template

Orchestration template for running a multi-agent architecture squad. Use this to sequence agents, validate handoffs, and trap assumptions at each stage.

---

## Purpose

When running a squad of architecture agents in sequence (Requirements Analyst → Architecture Explorer → Adversarial Reviewer → Documentation Generator), coordination ensures:
- Clear handoff gates (no silent failures)
- Assumption validation at each step
- Confidence scores tracked and escalated when <3
- Validation failures caught before cascading to later agents

---

## Pre-Squad Checklist

**Before launching agents, verify:**

- [ ] Project context documented (domain, business drivers, constraints)
- [ ] Stakeholder requirements available (or initial brief ready)
- [ ] Constraints inventory drafted (team size, budget, deployment, latency, compliance, timeline)
- [ ] Success criteria defined (architecture decision criteria, approval gates)
- [ ] Squad roles assigned (Requirements Analyst, Explorer, Reviewer, Documentation lead)
- [ ] Shared templates confirmed in place (`/shared/adr.instructions.md`, `/shared/assumption.instructions.md`, etc.)
- [ ] Escalation contacts identified (CTO for conflicts; stakeholders for requirement clarification)

---

## Squad Execution Phases

### Phase 1: Requirements Analyst — Clarify & Register Assumptions

**Agent Responsibility:**
- Extract functional + non-functional requirements from stakeholder input
- Identify ambiguities and escalation points
- Draft Assumptions Register (confidence 1–5 on each assumption)
- Output ASSUMPTIONS REGISTER table for next agent to validate

**Handoff Validation Checklist (from Analyst):**
- [ ] [INPUT MISSING] fail-fast checks completed (all required inputs present)
- [ ] Functional requirements numbered and traced to explicit input (no invented)
- [ ] Non-functional requirements have numeric targets (not "fast," "scalable")
- [ ] Every assumption marked with origin (EXPLICIT/INFERRED/SPECULATIVE) + confidence (1–5)
- [ ] Ambiguities phrased as explicit questions for stakeholders
- [ ] [CONFIDENCE SCORE] provided: overall confidence on requirements analysis (1–5)
- [ ] No assumptions <2/5 unless flagged for human validation

**Escalation Rules for Analyst:**
- If requirements contradict (e.g., "6-month launch" + "100% test coverage"), output `[VALIDATION FAILED: <reason>]` and escalate to stakeholder
- If critical NFR target missing (e.g., "fast" with no ms target), output `[INPUT MISSING: <field>]`
- If constraint is vague (e.g., "large scale" without numbers), output `[CLARIFICATION NEEDED]`

**Phase 1 Outcome:** ASSUMPTIONS REGISTER + REQUIREMENTS SPEC + confidence score

---

### Phase 2: Architecture Explorer — Generate Options & Validate Assumptions

**Agent Responsibility:**
- Accept Requirements + Assumptions Register from Phase 1
- Generate 3–5 distinct architecture options that fit all constraints
- For each option: validate assumptions from Phase 1 (ACCEPTED/FAILED/NEEDS CLARIFICATION)
- Create new assumptions if generating options (marked with origin + confidence)
- Output tradeoff table with cost, latency, ops complexity, risk
- Output overall [CONFIDENCE SCORE] on all architectures

**Input Validation (Explorer checks before proceeding):**
```
VALIDATION CHECKLIST FOR ARCHITECTURE EXPLORER:
- [ ] Assumptions Register from Phase 1 includes confidence scores (1–5)
- [ ] Functional requirements are specific (not invented)
- [ ] NFRs have numeric targets
- [ ] No assumptions marked [VALIDATION FAILED] from Phase 1
- [ ] If ANY assumption <3/5, has human confirmed (or I will escalate)
```

**Handoff Validation Checklist (from Explorer):**
- [ ] All stated constraints from Phase 1 respected in every option (or explicitly marked VIOLATES)
- [ ] Cost estimates reference specific services (not "low"/"medium"/"high")
- [ ] Assumptions from Phase 1 explicitly addressed (ACCEPTED/FAILED/NEEDS CLARIFICATION table)
- [ ] New assumptions created during option generation marked with confidence + origin
- [ ] Tradeoff table uses numeric metrics with [GROUNDED: <source>] tags
- [ ] All confidence scores <3 flagged and marked for validation by next agent
- [ ] [CONFIDENCE SCORE] provided on architecture options overall (1–5)

**Escalation Rules for Explorer:**
- If all viable options violate a constraint (impossible brief), output `[VALIDATION FAILED: impossible brief]` and escalate
- If assumptions from Phase 1 contradict my findings, output `[VALIDATION FAILED: <contradiction>]`
- If I cannot provide grounded reasoning (confidence <2), escalate to human instead

**Phase 2 Outcome:** ARCHITECTURE OPTIONS (3–5) + ASSUMPTIONS VALIDATION TABLE + TRADEOFF TABLE + confidence score

---

### Phase 3: Adversarial Reviewer — Challenge & Risk Analysis

**Agent Responsibility:**
- Accept architecture options + assumptions from Phase 2
- For each architecture, identify failure modes, hidden risks, edge cases
- Challenge assumptions that seem unsupported
- Validate that trade-off analysis is complete
- Output risk register + recommendations
- Output [CONFIDENCE SCORE] on architecture selections

**Input Validation (Reviewer checks before proceeding):**
```
VALIDATION CHECKLIST FOR ADVERSARIAL REVIEWER:
- [ ] Phase 1 Assumptions Register present + confidence scores visible
- [ ] Phase 2 Architecture options each have constraint compliance table (FITS/VIOLATES/NEEDS VALIDATION)
- [ ] Phase 2 Assumptions Validation table present (ACCEPTED/FAILED/NEEDS CLARIFICATION)
- [ ] Tradeoff analysis complete (cost, latency, ops complexity per option)
- [ ] No [VALIDATION FAILED] markers unresolved from Phase 1 or 2
```

**Handoff Validation Checklist (from Reviewer):**
- [ ] Each architecture has documented failure modes (>3 scenarios per option)
- [ ] Risks are specific (not "complexity may be high")
- [ ] Trade-off analysis challenged (e.g., "you claimed low ops complexity; here's why that's wrong")
- [ ] Assumptions questioned (confidence justified or escalated)
- [ ] Hidden risks identified (team skill gaps, vendor lock-in, scaling cliffs)
- [ ] Recommendations include contingency plans for high-risk assumptions
- [ ] [CONFIDENCE SCORE] provided on architecture recommendations (1–5)

**Escalation Rules for Reviewer:**
- If assumption from Phase 2 seems unjustified (confidence claimed 4/5 but no evidence), escalate to Explorer + human
- If architecture violates requirement that wasn't caught in Phase 1–2, output `[VALIDATION FAILED: <violation>]`
- If risk mitigation requires team learning/hiring, escalate to stakeholder for approval

**Phase 3 Outcome:** RISK REGISTER + RECOMMENDATIONS + FAILURE MODE ANALYSIS + confidence score

---

### Phase 4: Documentation Generator — Package & Finalize

**Agent Responsibility:**
- Accept final architecture choice + all assumptions + risks from Phase 1–3
- Generate ADR (Architecture Decision Record) documenting choice + rationale
- Generate process flows for deployment, operations, incident response
- Generate running Assumption Register for ongoing monitoring
- Output summary for stakeholders

**Input Validation (Documentation checks before proceeding):**
```
VALIDATION CHECKLIST FOR DOCUMENTATION GENERATOR:
- [ ] Final architecture choice selected + approved by lead
- [ ] All [VALIDATION FAILED] and [CONFIDENCE ALERT] markers resolved
- [ ] Assumptions Register consolidated from all phases (A1–A4)
- [ ] Risk Register complete with mitigations
- [ ] Tradeoff rationale clear (why this option won vs. others)
```

**Handoff Validation Checklist (from Documentation):**
- [ ] ADR includes: Decision, Context, Options Considered, Consequences, Assumptions
- [ ] Process flows documented (deployment, scaling, incident response, rollback)
- [ ] Running Assumption Register includes: ID, Statement, Origin, Confidence, Validation Plan, Deadline, Owner
- [ ] Risk Register includes: Risk, Impact, Likelihood, Mitigation, Owner, Review Date
- [ ] Stakeholder summary clear (1-page executive summary)

---

## Validation Handoff Template

**When Agent N hands off to Agent N+1, use this:**

```markdown
## HANDOFF from [Agent Name] to [Next Agent Name]

**My outputs:**
- [List primary deliverables: options, risk analysis, etc.]
- [List key assumptions I created]
- [List confidence scores and escalations]

**VALIDATION CHECKLIST FOR YOU ([Next Agent]):**
- [ ] My assumptions: do you accept all confidence scores? (If not: respond [VALIDATION FAILED: <reason>])
- [ ] My outputs: contradictions or gaps? (If yes: respond [VALIDATION FAILED: <contradiction>])
- [ ] Confidence <3 on critical items? (If yes: respond [CONFIDENCE ESCALATION: <items>])

**PROCEED ONLY IF:**
- You respond [VALIDATION ACCEPTED], OR
- You resolve all <3-confidence items with human first

**If you respond [VALIDATION FAILED]:**
- Explain the issue
- I will regenerate with corrections
- We loop back to Phase [X]

---
```

---

## Confidence Score Escalation Rules

**When confidence <3 appears at any phase:**

1. **Phase 1 (Analyst):** If any assumption <3/5, flag [CONFIDENCE ALERT: <assumption>]. Explorer must validate with human before proceeding.

2. **Phase 2 (Explorer):** If any architecture option <3/5, flag [CONFIDENCE ALERT: <architecture>]. Reviewer must validate with human before proceeding.

3. **Phase 3 (Reviewer):** If risk mitigation confidence <3/5, flag [HIGH_RISK: <mitigation>]. Must be approved by CTO before proceeding to Phase 4.

4. **Phase 4 (Documentation):** If running Assumption Register contains items <3/5 AND not yet validated, flag [VALIDATION NEEDED] in ADR. Team must establish validation checkpoints before launch.

---

## Conflict Resolution During Squad Execution

| Conflict | Resolution |
|----------|-----------|
| Explorer rejects Phase 1 assumptions; Analyst disagrees | Escalate to human stakeholder for clarification; both agents loop back to Phase 1 with corrected inputs |
| Reviewer finds risk that Phase 2 missed; Explorer says risk acceptable | Document disagreement in Risk Register; mark as [ESCALATE TO CTO] |
| Phase 3 recommendation contradicts Phase 1 constraint | Output [VALIDATION FAILED: <contradiction>]; loop back to Phase 1 for constraint re-validation |

---

## Squad Execution Timeline

| Phase | Agent | Duration | Input | Output | Gate |
|-------|-------|----------|-------|--------|------|
| 1 | Requirements Analyst | 1–2 days | Stakeholder brief | Requirements + Assumptions Register (confidence 1–5) | No <3-confidence assumptions unresolved |
| 2 | Architecture Explorer | 2–3 days | Phase 1 output | Options + Tradeoff analysis + assumption validation | No [VALIDATION FAILED] markers |
| 3 | Adversarial Reviewer | 1–2 days | Phase 2 output | Risk register + recommendations + challenge notes | All high-risk assumptions escalated |
| 4 | Documentation Generator | 1 day | Phase 3 output | ADR + process flows + running assumption register | ADR approved by lead; [VALIDATION NEEDED] items assigned owners |

**Total timeline:** 5–8 business days for complete architecture package

---

## Squad Monitoring Dashboard

**Track throughout execution:**

```
Phase | Agent | Status | Confidence | Escalations | Blockers
------|-------|--------|------------|-------------|----------
1 | Analyst | ✓ Done | 4/5 | [CLARIFY: deployment env] | None
2 | Explorer | 🔄 In Progress | 3/5 | [CONFIDENCE: team learning curve] | Waiting on Phase 1 clarification
3 | Reviewer | ⏳ Not Started | — | — | Blocked on Phase 2
4 | Documentation | ⏳ Not Started | — | — | Blocked on Phase 3
```

---

## Post-Squad Checklist

**After Phase 4 (before finalizing architecture):**

- [ ] Final ADR approved by Lead Architect + CTO
- [ ] Running Assumption Register assigned owners + validation deadlines
- [ ] Risk Register owners acknowledged + mitigation plans in place
- [ ] Process flows (deployment, incident response) reviewed by SRE
- [ ] Stakeholder summary presented + approved
- [ ] Team ready for implementation (backlog prioritized, dependencies clear)

---

## Common Pitfalls & Prevention

| Pitfall | Prevention |
|---------|-----------|
| Agents skip [INPUT MISSING] checks → hallucinations cascade | Enforce fail-fast validation; escalate immediately if inputs missing |
| Confidence scores not justified → inflated 4–5/5 despite weak reasoning | Require explicit justification for every confidence score; challenge <3 and >4 scores |
| Assumptions created in Phase 2–3 not added to running register | Consolidate all assumptions from all phases into Phase 4 register; audit for completeness |
| Handoffs happen without validation → errors not caught until implementation | Require explicit [VALIDATION ACCEPTED] gate before proceeding to next phase |
| Escalations unresolved → team waits | Assign escalation owner + 24-hour response deadline for clarifications |

---

## Templates Referenced

- **Phase 1:** `/shared/assumption.instructions.md` (Assumptions Register format)
- **Phase 2:** `/shared/architecture-specifications.instructions.md` (architecture spec template)
- **Phase 3:** `/shared/risk-document.instructions.md` (risk register format)
- **Phase 4:** `/shared/adr.instructions.md` (ADR format) + `/shared/process-flow.instructions.md` (process flows)

---

**Last Updated:** 2026-06-02
