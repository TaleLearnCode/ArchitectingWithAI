# AI-Augmented Architecture Prompt Library

**For architects who collaborate responsibly with AI.**

---

## Purpose & Audience

This library provides copy-paste-ready prompts and workflows for architects using AI as a thinking partner. It operationalizes the session thesis: **AI broadens exploration; architects supply judgment.**

**You should use this library if you:**
- Need to clarify ambiguous requirements before designing
- Want to explore multiple architecture options with justified tradeoffs
- Require adversarial review to expose blind spots
- Generate ADRs, process flows, and architecture specs from structured decisions
- Orchestrate multi-agent AI workflows for complex architecture problems
- Want safeguards against hallucinations, invented constraints, and overconfidence

**You should NOT use this library if you:**
- Are building brainstorming artifacts (not decisions)
- Want generic architecture templates (this is specific to your constraints)
- Expect AI to replace architect judgment (it won't)

---

## Recommended Section Progression

The library progresses through a real architecture workflow:

```
01 → 02 → 03 → 04 → (05 optional) → 06
 ↑                                     ↓
 └─────── Feedback loops ─────────────┘
```

**Phase 1 (30 minutes):** 01 + 02  
**Phase 2 (1 hour):** 01 + 02 + 03 + 04  
**Phase 3 (2+ hours):** Full workflow + Squad + Governance  

---

## Section Index

### **Section 01 — Requirements Clarification**
**Intent:** Uncover ambiguities, hidden constraints, and missing non-functional requirements before design begins.

Prompts help you:
- Scan requirements for contradictions and gaps
- Extract implicit non-functional requirements (performance, security, compliance)
- Simulate stakeholder perspectives (Product, Security, SRE, Finance, Compliance)
- Generate clarifying questions architects must answer first

**When to use:** Immediately after requirements are handed off.  
**Workflow time:** 10–15 minutes with AI + 5–10 minutes reviewing output.  
**Key guardrail:** Flag inferred content; do not invent NFRs.

---

### **Section 02 — Architecture Exploration**
**Intent:** Widen the solution space and force tradeoff justification instead of premature optimization.

Prompts help you:
- Generate constraint-driven architecture options (not the same 4 every time)
- Create tradeoff tables with grounded rationale, not arbitrary 1–5 scores
- Generate Quality Attribute Scenarios for your specific system
- Eliminate non-viable patterns based on explicit constraints

**When to use:** After requirements are clarified; before designing.  
**Workflow time:** 15–20 minutes generating options + 15–20 minutes reviewing/debating tradeoffs.  
**Key guardrail:** Every option must connect to explicit constraints; rejected patterns require explanation.

---

### **Section 03 — Adversarial Review**
**Intent:** Attack your design to expose hidden assumptions, scaling risks, and failure modes.

Prompts help you:
- Conduct brutal architecture reviews without hiring consultants
- Simulate an Architecture Review Board with real roles (CTO, CISO, SRE, Principal Architect, Finance)
- Analyze catastrophic failure scenarios (what would break this?)
- Identify unverified assumptions and integration risks

**When to use:** After you have a leading architecture option; before finalizing decisions.  
**Workflow time:** 15–20 minutes attack + 20–30 minutes addressing findings.  
**Key guardrail:** Criticism must be concrete (not "this might fail"); base on known failure patterns.

---

### **Section 04 — Documentation Generation**
**Intent:** Generate ADRs, process flows, and risk registers from your architecture decisions.

Prompts help you:
- Create Architecture Decision Records with context, options, and consequences
- Document multi-step processes with explicit actors, timing, and error handling
- Generate Architecture Specifications with component responsibilities and interfaces
- Build Risk Registers and Assumption Registers with validation/fallback plans

**When to use:** After architecture is decided and reviewed; to formalize decisions.  
**Workflow time:** 10–15 minutes per document (ADR, Process Flow, Risk entry).  
**Key guardrail:** Use inline templates; do not reference external files attendees may not have.

---

### **Section 05 — Multi-Agent Workflows**
**Intent:** Orchestrate multiple AI agents to work through a full architecture problem in sequence.

Prompts help you:
- Coordinate Requirements Analyst → Architecture Options → Tradeoff Analyst → Adversarial Reviewer → Documentation agents
- Ensure each agent validates prior work before adding its contribution
- Prevent hallucinations from cascading through the pipeline

**When to use:** For complex problems requiring 2+ hour workflows or team collaboration simulation.  
**Workflow time:** 45–120 minutes depending on scope.  
**Key guardrail:** Each handoff includes validation gates (confidence scores, assumption registers, flags for speculation).

---

### **Section 06 — Governance & Anti-Slop**
**Intent:** Prevent AI from generating bullshit and catch hallucinations before they enter decisions.

Prompts help you:
- Remove redundant filler and overly broad claims
- Flag speculative content ([INFERRED], [SPECULATIVE], [UNKNOWN])
- Validate outputs using practical checklists
- Enforce document boundaries (ADRs stay ADRs; process flows stay process flows)

**When to use:** After every AI output; before trusting the result.  
**Workflow time:** 5–10 minutes validation per artifact.  
**Key guardrail:** Treat flagged content as uncertain until you verify it.

---

## How to Use This Library in a Real Architecture Workflow

### Quick Start (30 minutes)

1. **Paste your requirements** into Section 01 prompt
2. **Review the output** — look for [INFERRED] flags and clarify gaps with stakeholders
3. **Paste clarified requirements** into Section 02 prompt
4. **Review architecture options** — which ones fit your team, budget, and constraints?
5. **Pick a leading option** — save it for Section 03

### Standard Workflow (1 hour)

1. **Follow Quick Start** (steps 1–5)
2. **Paste your leading architecture** into Section 03 prompt (Adversarial Review)
3. **Read the attack carefully** — what assumptions does it challenge?
4. **Adjust architecture** if needed; re-run Section 03 if major changes
5. **Paste final architecture** into Section 04 prompt (Documentation Generation)
6. **Generate an ADR** for the key decision; review for hallucinations using Section 06

### Comprehensive Workflow (2+ hours with Squad)

1. **Follow Standard Workflow** (steps 1–6)
2. **Spawn a Squad** (Section 05) with 5 agents working in sequence
3. **Validate each handoff** against the checklist in the handoff prompt
4. **Compile final architecture package** — Summary, ADRs, Process Flows, Risks, Assumptions
5. **Run full governance pass** (Section 06) — remove slop, validate all outputs

### Customization by Project Type

All sections include **if-then matrices** at the top for common constraints:
- **Team size:** 3–5 / 6–15 / 15+
- **Domain:** B2B SaaS / Real-Time / Regulated / Monolithic Legacy
- **Operational maturity:** Early / Mature / Enterprise

Pick your row; adjust prompt parameters accordingly.

---

## Build Order Recommendation for Section-by-Section Expansion

**Priority order for content completion:**

| Order | Section | Effort | Risk | Dependencies |
|-------|---------|--------|------|--------------|
| 1 | 06 Governance | Low | Medium | None—start with guardrails |
| 2 | 01 Requirements | Low | Low | Governance applied |
| 3 | 02 Architecture | Medium | High | 01 complete; guardrails applied |
| 4 | 04 Documentation | Low | Low | Templates inlined; 02 complete |
| 5 | 03 Adversarial | Medium | Medium | 02 complete; example failures curated |
| 6 | 05 Multi-Agent | High | High | All others complete; validation gates proven |

**Rationale:**
- **Start with Section 06 (Governance):** Establish guardrails first; apply to all downstream prompts. Prevents hallucination risk from the start.
- **Build 01 & 02 (Requirements + Architecture):** These are the spine. Once solid, other sections flow naturally.
- **Add 04 (Documentation):** Low risk; enables ADR/Process Flow generation immediately after 02.
- **Deepen 03 (Adversarial):** Requires curated failure examples; medium complexity.
- **Save 05 (Multi-Agent) for last:** Highest risk (cascade failures); only deploy after 1–4 are production-hardened.

**Quality gates before each release:**
- Governance checklist applied to every prompt
- Guardrails tested with seeded-bad requirements (must catch hallucinations)
- Examples include before/after (good output, weak output, remediation)
- Personas grounded in explicit goals/KPIs, not generic
- Scales grounded in constraints, not arbitrary

---

## Thesis Integration: Every Prompt Reinforces "AI Broadens, Architects Judge"

**In Section 01 (Requirements):** AI surfaces gaps and contradictions; *you* decide what to clarify first.

**In Section 02 (Architecture):** AI widens the solution space; *you* judge which options fit your constraints, team, and budget.

**In Section 03 (Adversarial Review):** AI attacks your assumptions; *you* decide whether to adapt or defend.

**In Section 04 (Documentation):** AI drafts; *you* verify for accuracy and tailor to your context.

**In Section 05 (Multi-Agent):** AI coordinate; *you* validate each handoff and catch errors early.

**In Section 06 (Governance):** AI generates; *you* enforce discipline (no bullshit, no invented constraints).

---

## What's Next?

- **Pick a real requirement** from your current project
- **Start with Section 01** and give it 15 minutes
- **Review output for [INFERRED] and [SPECULATIVE] flags**
- **Proceed to Section 02** if requirements clarification is solid
- **Share feedback** — guardrails working? Prompts too verbose? Examples helpful?

---

**Last Updated:** 2026-06-01  
**Audience:** Architects learning to collaborate with AI  
**Prerequisite:** Familiarity with ADRs, architecture tradeoffs, and constraint-driven design
