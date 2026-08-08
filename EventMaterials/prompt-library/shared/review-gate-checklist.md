# Prompt Library Review Gate Checklist

**Author:** Switch (Critic/Tester)  
**Purpose:** Quality gate for publishing each prompt section to prevent predictable failures  
**Applied To:** All 6 sections before distribution  
**Status:** Required; blocks publication until all blocking criteria pass

---

## Overview

This gate is applied **section-by-section** before publishing. A prompt section must pass ALL blocking criteria to proceed. Major issues should be corrected before resubmission; non-blocking recommendations may be tracked for future improvement.

**Publication Flow:**
1. Section author submits prompts with supporting materials
2. Reviewer (Switch) runs this checklist
3. If blocking criteria fail → Reject; reassign to different author for revision
4. If major criteria fail → Approved with Required Changes
5. If all criteria pass → Approved; ready for publication

---

## BLOCKING CRITERIA (Red Lights)

These must pass or the section is rejected.

### 1. Vague Inputs — Explicit Prerequisites Defined

**Criterion:** Every prompt explicitly lists required inputs; prompts fail gracefully if inputs are missing.

**What to check:**

- [ ] **Every prompt has a "Prerequisites" section** identifying required data inputs (not optional suggestions)
  - Example: "User stories with acceptance criteria (required; 5+ representative stories)"
  - NOT: "Have requirements handy (optional)"

- [ ] **Validation rule is explicit:** Prompt includes a statement like: "If [input] is missing, output `[INPUT MISSING: <name>]` and stop."

- [ ] **No "all X" inferential statements** without flags
  - ❌ "Extract all implicit non-functional requirements"
  - ✓ "Extract ONLY explicit non-functional requirements. If you must infer, mark as `[INFERRED]`"

- [ ] **Input format is specified** (e.g., "User stories with 'As a / I want / So that' format")

- [ ] **Validation applied:** Test one prompt with deliberately missing input; confirm it outputs `[INPUT MISSING]` instead of hallucinating

**Failure consequence:** Attendees provide half-formed requirements and AI invents the rest.

**Remediation:** Add prerequisite checklist; add fail-fast validation statement; test with broken input.

---

### 2. Ungrounded Scoring — Ratings Reference Explicit Constraints

**Criterion:** Any numerical or subjective scale (1–5, high/medium/low, cost/benefit) must be tied to actual constraints from input.

**What to check:**

- [ ] **No scale without grounding rule**
  - ❌ "Rate these architectures 1–5 on scalability"
  - ✓ "Rate these architectures 1–5 on scalability. For each rating, cite which constraint it is based on (e.g., 'Monolith: 2/5 on scalability because constraint says 100K concurrent users and monoliths typically plateau at 50K'). If the constraint is unknown, output `[UNKNOWN]` instead of guessing."

- [ ] **Confidence scores included** (if any major recommendation is made, AI states confidence 1–5 with reasoning)

- [ ] **All ratings are traced in output** — Not: "Option A: 4/5 cost-effective" But: "Option A: 4/5 cost-effective [GROUNDED: assumes no specialized DBAs required; cost = infrastructure + ops labor; given 6-person team with no DBA specialist, monolith is lower total cost]"

- [ ] **No arbitrary benchmarks** — If using a scale (e.g., "must be < 100ms latency"), that scale is from input, not invented
  - Test: Could I trace every score back to something the attendee provided? If not, it's invented.

- [ ] **Validation applied:** Extract one scoring section; verify every score references a specific input constraint; if it doesn't, flag it

**Failure consequence:** Attendees see "Kubernetes: 5/5" and think it's data-driven; it's actually arbitrary.

**Remediation:** Add explicit traceability rule to scoring prompts; embed constraint list in prompt output; use `[GROUNDED]` / `[UNKNOWN]` tagging.

---

### 3. Hallucination Cascade Risk — Multi-Agent Pipelines Have Validation Gates

**Criterion:** If any section uses multi-agent workflows (e.g., Agent 1 → Agent 2 → Agent 3), each handoff must include validation gates.

**What to check:**

- [ ] **Each agent outputs `[ASSUMPTIONS]` list** (what did I assume that wasn't explicit?)

- [ ] **Each agent outputs `[CONFIDENCE SCORES]`** (how confident am I in each major claim? 1–5, with reasoning)

- [ ] **Next agent in pipeline includes validation checklist:**
  ```
  VALIDATION CHECKLIST (Required):
  1. [ ] Previous agent's [ASSUMPTIONS]: Are they validated by input? Flag unverified.
  2. [ ] [CONFIDENCE SCORES] < 3 on critical claims: Escalate or stop.
  3. [ ] [SPECULATIVE] content: Verify stated assumptions or flag [ASSUMPTION FAILED].
  4. [ ] Traceability: Can I trace every recommendation back to input (not invented)?
  
  If any check fails, output [VALIDATION FAILED] with specific issues. Do not proceed silently.
  ```

- [ ] **No silent handoffs** — Prompt must explicitly verify upstream before proceeding

- [ ] **Test scenario:** Intentionally include hallucination in Agent 1 output; verify Agent 2 catches it and escalates

- [ ] **ASSUMPTION REGISTER** (if section 5+ uses squads): Create a running table; each agent adds rows; downstream agents update Status (VALIDATED / UNVALIDATED / FAILED)

**Failure consequence:** Invented scale requirement cascades through 4 agents, resulting in over-architected solution.

**Remediation:** Add validation gates; require `[ASSUMPTIONS]` + `[CONFIDENCE SCORES]` in every agent output; include validation checklist in next agent's prompt.

---

### 4. Missing Customization Guidance — Real Constraint Matrices Present

**Criterion:** Every prompt includes explicit guidance on how to customize it for different project contexts.

**What to check:**

- [ ] **"Customization Knobs" section** present in every prompt (not just landing page)
  - Must include at least 3–4 knobs: Depth, Focus, Scope, Tone (or equivalent for the prompt type)
  - Each knob must include a concrete example modification

- [ ] **Constraint matrix or persona mapping** (if applicable)
  - If prompt recommends architecture options: Include a decision table like:
    ```
    | Team Size | Budget | SLA | Recommended Option | Why Not Others |
    |------------|--------|-----|-------------------|-----------------|
    | 3–5 ppl   | $100K  | 99% | Monolith + CDN    | Microservices too much ops overhead |
    | 20+ ppl   | $500K  | 99.9% | Microservices  | Monolith can't split load |
    ```

- [ ] **"When NOT to use this prompt" section** included
  - Example: "If requirements are still being written, use Requirements Ambiguity Scan first. If you've already chosen a technology, skip to Adversarial Review."

- [ ] **Iteration guidance** (if output fails validation checklist, what should attendee do?)
  - Fallback suggestions: "If this prompt gives generic output, try adding 'Focus on [team constraint]' or switch to [related prompt in Section X]"

- [ ] **Role/persona guidance** (if relevant)
  - Who should run this? Architect alone? With team? With stakeholders?
  - Time budget: 15 min, 1 hour, or 2 hours?

- [ ] **Validation applied:** Can a new team member customize this prompt for a different project size/budget without asking for help?

**Failure consequence:** Attendee runs template as-is; output doesn't match their context; they think the library is generic fluff.

**Remediation:** Add constraint matrix; add customization knobs with examples; add "when not to use" section; add role/timing guidance.

---

### 5. Missing Verification Loop — Validation Checkpoints Present

**Criterion:** Every prompt includes a validation checklist attendee runs before acting on output.

**What to check:**

- [ ] **Validation Checklist** section present (not just in template; in actual prompt)
  - Minimum 4–6 items, checkable without domain expertise
  - Must include: Traceability, Specificity, No Boilerplate, No Inventions, Assumption Flagging

- [ ] **Example checklist for this prompt type:**
  ```
  Before acting on this output, check:
  - [ ] Traceable to Input: Every recommendation references something I provided
  - [ ] Specific to My Project: Output changes if I change [key constraint]
  - [ ] No Generic Boilerplate: Phrases like "best practice" come with context
  - [ ] Assumptions Flagged: Inferred content marked [INFERRED] with reasoning
  - [ ] No Invented Data: All examples/metrics come from my input or known baselines
  
  Scoring: All checked ✓ = trustworthy. 1–2 unchecked = iterate. 3+ unchecked = major revision needed.
  ```

- [ ] **Iteration guidance included**
  - If checklist fails (2+ items), what should attendee do?
  - Example: "If 'Traceable to Input' fails, ask the AI: 'For each option, which specific constraint from my project supports this choice?'"

- [ ] **Fallback strategy** (if iterations don't help)
  - "Try switching AI platform (ChatGPT → Claude)"
  - "Try related prompt in Section [X]"
  - "Escalate to architect review"

- [ ] **Anti-patterns called out** (common failure modes with recovery steps)

- [ ] **Validation applied:** Run checklist on example output; confirm it catches weak output without false positives

**Failure consequence:** Attendee blindly trusts output; misses hallucinations; makes poor architecture decision based on AI nonsense.

**Remediation:** Add explicit validation checklist; add examples of good vs. weak output; add iteration guidance; add fallback strategies.

---

## MAJOR CRITERIA (Yellow Lights)

These should be corrected before publication. If 2+ major criteria fail, treat as blocker.

### 6. Anti-Slop Guardrails — Generic Boilerplate Removed

- [ ] **No vague hedging** ("may need to consider," "could be important")
- [ ] **No generic "best practices"** without context
- [ ] **Output is specific** to the constraint set provided
- [ ] **Repeated content eliminated** (prompts don't say the same thing twice)
- [ ] **Measured claims only** (ratings, evidence-based, not adjectives)

**Example weak:** "Microservices provide scalability and resilience."  
**Example strong:** "Microservices enable independent scaling per service, supporting your 100K concurrent user requirement. Trade-off: 60% more operational complexity for your 5-person team."

---

### 7. Template Dependencies Clear — All File References Checkable

- [ ] **If prompts reference external templates** (adr.instructions.md, process-flow.instructions.md), they are:
  - Either inlined in the prompt itself
  - Or documented with exact file path + validation that file exists
  - Or provided with fallback inline version

- [ ] **No broken file references** — All `.md` files referenced exist in the published directory

- [ ] **URLs to external resources** (if any) are tested and working

---

### 8. Examples Real, Not Invented — Good + Weak Outputs Provided

- [ ] **At least 2 examples per prompt:**
  - 1 good example (shows strong output)
  - 1 weak example (shows common failure)

- [ ] **Examples are realistic**, not toy problems
  - Good: "E-commerce, 6-person team, $100K budget, 6-month deadline"
  - Bad: "Generic web app"

- [ ] **Weak example explains WHY it's weak** and how to fix it

---

### 9. Thesis Integration — Prompts Reinforce Project Narrative

- [ ] **Section relates back to core thesis:** "AI broadens exploration; architects supply judgment"

- [ ] **Prompt language reflects this** 
  - Not: "Let AI decide your architecture"
  - But: "Use AI to explore options you might have missed; then decide which fits your constraints"

---

### 10. Cross-Section Sequencing — Prerequisite Prompts Completed First

- [ ] **If Section 2 (Architecture Exploration) references outputs from Section 1 (Requirements), Section 1 is published first**

- [ ] **No circular dependencies** between sections

- [ ] **Recommended build order followed:**
  1. Section 1 (Requirements Clarification)
  2. Section 6 (Governance / Anti-Slop)
  3. Section 2 (Architecture Exploration)
  4. Section 4 (Documentation Generation)
  5. Section 3 (Adversarial Review)
  6. Section 5 (Multi-Agent Squad)

---

## NON-BLOCKING RECOMMENDATIONS (Info)

These are nice-to-haves; track for Phase 2+ enhancement:

- Input validation prompts (pre-flight checklist before main prompt)
- Output scoring rubric (predefined grading scale for AI recommendations)
- Cost/ROI analysis templates
- Technology research & vetting prompts
- Team collaboration workflow guidance

---

## HOW TO USE THIS GATE

### For Section Authors

**Before submitting for review:**

1. Open this checklist
2. Go through **Blocking Criteria** (sections 1–5)
3. Confirm every item is checked
4. Ask: "Would a stranger be able to run these prompts and verify output quality?"
5. If any item is unchecked, revise before submission

### For Reviewers (Switch)

**When reviewing a section:**

1. Request the prompt files + any supporting materials (customization matrices, examples)
2. Run Blocking Criteria checklist manually
3. If any blocker fails:
   - Reject the section
   - Reassign to different author (enforce lockout from reviewer protocol)
   - Document specific blocker in rejection message
4. Run Major Criteria checklist
   - If 2+ major criteria fail, escalate to blocker status
5. If all criteria pass → Approved

**Testing validation gates (for multi-agent sections):**
- Inject hallucination into mock Agent 1 output
- Verify Agent 2 validation checklist catches it
- Confirm flow escalates instead of silently proceeding

### For Team Coordination

- **Section publication requires Review Gate approval**
- **No section proceeds without Switch sign-off**
- **Rejected sections reassigned per Reviewer Protocol** (original author locked out)

---

## EXAMPLE APPLICATION

### Scenario: Reviewing Section 2 (Architecture Exploration)

**Section submits:** 3 prompts (Generate Options, Tradeoff Table, Constraint-Aware Architecture)

**Blocking Check #1: Vague Inputs**
- ✓ "Generate Options" prompt lists: "Team size, concurrent users, budget, deployment location, SLA required"
- ✓ Validation rule included: "If any constraint is unknown, output `[UNKNOWN]` and explain why"
- ✓ Tested with missing input; confirmed it outputs `[UNKNOWN]` instead of guessing

**Blocking Check #2: Ungrounded Scoring**
- ❌ "Tradeoff Table" prompt says "Rate 1–5 on scalability, cost, etc." but doesn't reference constraints
- ❌ No instruction to trace ratings back to input constraints
- **BLOCKER TRIGGERED** → Reject; request revision

**Reviewer feedback:**
"Tradeoff ratings must be grounded. Add rule: 'For each rating, output [GROUNDED: <constraint>] or [UNKNOWN]. If constraint is missing, do not guess.'"

---

## RELATED ARTIFACTS

- **Prompt Guardrails Skill:** Patterns for preventing specific hallucinations
- **Multi-Agent Validation Skill:** Protocol for validation gates in squads (Section 5)
- **Prompt Template:** Baseline structure for all prompts
- **Customization Playbook:** How to adapt prompts for different contexts

---

## FAQ

**Q: What if a prompt doesn't fit all criteria?**  
A: Blocking criteria are non-negotiable. If a prompt genuinely doesn't need prerequisites (e.g., pure brainstorming), document that exception explicitly and get team consensus before publishing.

**Q: Who enforces the gate?**  
A: Switch (Critic/Tester) enforces all gates. Rejections reassign to different author per Reviewer Protocol.

**Q: Can I publish an incomplete section?**  
A: No. All prompts in a section must pass the gate before any are published. Partial publication creates inconsistent quality.

**Q: What if we discover a blocker after publication?**  
A: Pull the section immediately. Root-cause in post-event retrospective. Implement improvement in next release.

---

## Version

- **v1.0** — 2026-06-01
- **Status:** Active; required for Section 1–6 publication
