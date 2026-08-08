# Prompt Quality Rubric

*Acceptance checks to detect weak outputs, hallucinations, and AI slop before you act on it.*

---

## Overview

The prompt library is designed to produce high-quality outputs, but AI can still hallucinate, over-generalize, or confabulate. This rubric provides concrete checks to validate output quality **before you act on it**.

**Key Principle:** The cost of catching a bad recommendation is zero. The cost of acting on a hallucinated constraint is high.

---

## How to Use This Rubric

### Quick Path (3 minutes)
Run the **Fast Checks** section for any prompt output. If output fails fast checks, stop and rerun with clarification.

### Thorough Path (10–15 minutes)
Use the **Section-Specific Rubrics** if output will influence a major decision (architecture selection, constraint interpretation, risk mitigation).

### Expert Path (Optional)
Review **Guardrails & Evidence** if you have time and want to catch subtle issues.

---

## Fast Checks (Run First)

### 1. Traceability Check: Is This Grounded?

**Question:** Can I trace every claim back to something I told the prompt?

**How to Check:**
- Pick 3 major claims from the output
- For each claim, ask: "Did I provide this input, or did AI invent it?"
- Mark claims as [INPUT], [INFERRED], or [INVENTED]

**Example:**

| Claim | Source | Status |
|-------|--------|--------|
| "Microservices better for scaling" | AI synthesized this from your "high scalability" requirement | ✓ GROUNDED |
| "Use Kubernetes for orchestration" | You never mentioned Kubernetes; AI generated this | ✗ INFERRED |
| "Team has 5 senior engineers with 10 years experience each" | You said "5 engineers"; AI invented "senior" and "10 years" | ✗ INVENTED |

**Action:**
- ✓ GROUNDED claims: Acceptable
- ✗ INFERRED claims: Mark these [INFERRED] and decide if they're helpful or problematic
- ✗ INVENTED claims: This is hallucination. **Stop. Rerun with clarification.**

---

### 2. Confidence Check: Is AI Sure?

**Question:** Does AI hedge or confidently state things it shouldn't?

**How to Check:**
- Scan for confident statements with no basis
- Look for patterns: "Obviously," "clearly," "must," "always," "impossible"
- These words are hallucination signals when not grounded in your inputs

**Example:**

| Statement | Confidence Signal | Assessment |
|-----------|-------------------|------------|
| "This team obviously needs microservices because they're scaling to 1M users" | "obviously" + you DID say 1M users | ✓ OK (confident, grounded) |
| "Obviously you cannot use a monolith with this team size" | "obviously" + you DIDN'T state team size for this option | ✗ OVER-CONFIDENT |
| "You must use a database with ACID transactions" | "must" + no compliance requirements stated | ✗ INVENTED CONSTRAINT |
| "Serverless is impossible for real-time systems" | "impossible" + absolute statement | ✗ OVERGENERALIZATION |

**Action:**
- ✓ Confident & grounded: Acceptable (but still verify)
- ✗ Confident & ungrounded: **Add [UNVERIFIED] flag and rerun with constraints clarified**
- ✗ Overgeneralization: **Flag as [OVERGENERALIZATION] and ask for specific constraints**

---

### 3. Completeness Check: Is Anything Missing?

**Question:** Did AI answer my question, or did it deflect/generalize?

**How to Check:**
- Review what you asked for in the prompt
- Review what AI provided
- Look for gaps (sections marked N/A, TBD, or "depends on specific context")

**Example Gaps:**

| Your Question | AI Response | Assessment |
|---------------|-------------|------------|
| "Which architecture fits our $50K/month budget?" | Generic pros/cons; no cost estimate | ✗ INCOMPLETE |
| "How does this architecture handle our 10x traffic spike?" | "Depends on scaling strategy" | ✗ DEFLECTION |
| "What are the compliance implications?" | Generic "ensure compliance"; no specifics | ✗ HAND-WAVING |
| "What happens if the database fails?" | Lists 3 mitigation strategies with estimated MTTR | ✓ COMPLETE |

**Action:**
- ✓ Addresses your question directly: Acceptable
- ✗ Generic or deflects: **Rerun with specific scenarios and constraints added**
- ✗ Too abstract: **Ask for concrete examples and estimates**

---

## Section-Specific Rubrics

Use these rubrics for major deliverables (architecture options, ADRs, risk assessments).

---

### REQUIREMENTS CLARIFICATION PROMPTS

**Typical Output:** List of ambiguities, implicit NFRs, stakeholder concerns, clarifying questions.

#### Hallucination Risk Profile

| Risk | Signal | Fix |
|------|--------|-----|
| **Inventing requirements** | "The system probably needs real-time consistency" (you didn't mention this) | Flag [INFERRED]; ask: Does your business model require real-time? |
| **Over-interpreting** | "Since you mentioned APIs, you must need GraphQL" | API communication ≠ GraphQL requirement. Reject or ground in actual constraint. |
| **Missing obvious questions** | 5 questions provided; obvious critical gaps | Rerun: "Add questions about: X, Y, Z" |
| **Stakeholder projection** | "The CFO will worry about cost" (all architects worry about cost) | Generic persona simulation. Add specific KPIs: "CFO's goal is revenue optimization, constrained by audit trail cost" |

#### Quality Rubric

- [ ] **Ambiguities**: All listed ambiguities are actually ambiguous (not invented contradictions)
- [ ] **Implicit NFRs**: NFRs extracted from functional requirements; marked [INFERRED] if inferred
- [ ] **Stakeholder Questions**: Each question is grounded in a specific stakeholder KPI, not generic
- [ ] **No Invented Requirements**: Output doesn't state "you probably need X" without basis
- [ ] **Completeness**: Cover all stakeholder groups you named (Product, Security, SRE, Finance, Compliance, Operations)
- [ ] **Actionability**: Each clarifying question has a follow-up you can actually ask (not "tell us everything")

#### Acceptance Threshold

**PASS:** 5–6 items checked  
**FAIL:** 2+ items unchecked; rerun with: "Ground every inference. Mark assumptions [INFERRED]. For each stakeholder, reference their KPI."

---

### ARCHITECTURE EXPLORATION PROMPTS

**Typical Output:** Multiple architecture options with trade-offs, ratings, justifications.

#### Hallucination Risk Profile

| Risk | Signal | Fix |
|------|--------|-----|
| **Generic Options** | Same 4 options (monolith/microservices/event-driven/serverless) for every problem | Real-world: some options are unviable. Rerun: "Reject options that don't fit constraints; explain why." |
| **Meaningless Ratings** | "Scalability: 4/5" with no basis | Ground every rating: "Scalability: 4/5 because [constraint], assuming [assumption]" |
| **Invented Trade-offs** | "Microservices are cheaper" (contradicts your $50K budget context) | Rates can't contradict constraints. Rerun with constraints emphasized. |
| **Missing Constraints** | Ignores the team-size or budget constraint you provided | Output should reference every constraint. If not mentioned, ask why. |
| **Optimistic Fallacy** | Best-case scenarios only; no risk mitigation | Rerun: "For each option, explain the failure mode AND mitigation cost" |

#### Quality Rubric

- [ ] **Constraint-Driven Filtering**: Each option explicitly fits (or doesn't fit) your constraints
- [ ] **Viable Options Only**: 3–5 options provided; rejected options clearly explained
- [ ] **Grounded Ratings**: Every rating (1–5 scale) references a constraint or explicit assumption
- [ ] **Cost Transparency**: Each option includes estimated cost; grounded in market data
- [ ] **Operational Burden**: Each option states on-call complexity, incident response time
- [ ] **No Invented Capabilities**: Doesn't claim tech can do something unless you know it can
- [ ] **Risk-Aware**: Each option includes failure mode and mitigation strategy
- [ ] **Traceability**: Output changes if constraints change (isn't static)

#### Acceptance Threshold

**PASS:** 6–8 items checked  
**FAIL:** 2+ items unchecked; rerun with: "Rate only viable options. Ground every rating in an explicit constraint. Explain why rejected options don't fit."

---

### ADVERSARIAL REVIEW PROMPTS

**Typical Output:** Hidden assumptions, scaling risks, security gaps, failure modes.

#### Hallucination Risk Profile

| Risk | Signal | Fix |
|------|--------|-----|
| **Invented Failure Scenarios** | "The database could implode" (no specific scenario) | Not a real failure mode. Ask: "What specific sequence of events causes this?" |
| **Generic Critique** | "Consider monitoring" (applies to every architecture) | Demand specificity: "What specific metric do you monitor? What threshold triggers an alert?" |
| **Missing Domain Knowledge** | For e-commerce, criticizes "eventual consistency" without understanding inventory model | Rerun with domain context: "Our inventory model allows X-minute propagation delay. Is this a problem?" |
| **Unactionable Recommendations** | "Ensure security" (not actionable) | Reject vague recommendations. Demand: "Specific control? How do you verify it?" |

#### Quality Rubric

- [ ] **Specific Failures**: Each failure mode has a concrete trigger (not generic "what if")
- [ ] **Grounded in Architecture**: Critiques reference specific components you're using
- [ ] **Domain-Aware**: Criticisms understand your domain's risk profile
- [ ] **Actionable**: Every concern has a specific mitigation (testable, implementable)
- [ ] **Honest**: Includes trade-offs (no perfect architectures); explains what you're accepting
- [ ] **Evidence-Based**: Criticisms grounded in documented patterns, not speculation
- [ ] **Verification Plan**: For each risk, explains how you'd know if it occurs
- [ ] **No Sacred Cows**: Doesn't protect recommended architecture from critique

#### Acceptance Threshold

**PASS:** 6–8 items checked  
**FAIL:** 2+ items unchecked; rerun with: "Be specific. Reference exact components. For each risk, state: trigger, impact, detection method, mitigation."

---

### DOCUMENTATION GENERATION PROMPTS (ADRs, Process Flows, Specs)

**Typical Output:** Structured document (ADR, process flow, spec) with context, options, decisions.

#### Hallucination Risk Profile

| Risk | Signal | Fix |
|------|--------|-----|
| **Invented Options** | ADR lists 5 options; you only considered 3 | Rerun: "Document ONLY options we actually evaluated. Do not invent alternatives." |
| **Unsupported Decisions** | "Decision: Microservices. Consequence: Costs rise." (contradicts your constraint) | Consequences must follow logically. Rerun with: "Consequences must be grounded in architecture, not invented." |
| **Missing Traceability** | Process flow doesn't reference decision that led to it | Process flows should trace back to architecture decision. Add: "Reference the ADR that justifies this process." |
| **Template Dependency Failure** | "Use ADR template" but template isn't included | Rerun: "Include full template inline; do not reference external files." |
| **Pseudo-Documentation** | "Document the security model" → "Ensure data is secure" (not a document) | Rerun: "Be specific. Document: [component] uses [mechanism] to protect [data]." |

#### Quality Rubric

- [ ] **Template Compliance**: Follows the structure you specified (ADR format, process flow format, etc.)
- [ ] **Options Grounded**: Options are ones you actually considered (not invented by AI)
- [ ] **Decision Justified**: Chosen option clearly wins on stated criteria
- [ ] **Consequences Realistic**: Consequences follow from decision (not invented)
- [ ] **Complete Content**: No N/A, TBD, or placeholder sections; document is ready to use
- [ ] **Traceable**: Document references decisions/requirements that justify it
- [ ] **No Invented Entities**: Doesn't reference processes, tools, or teams you don't have
- [ ] **Accuracy**: Can be reviewed by someone unfamiliar with the project and understood
- [ ] **Timestamp/Author**: If required format, includes metadata

#### Acceptance Threshold

**PASS:** 7–9 items checked  
**FAIL:** 2+ items unchecked; rerun with: "This is a [document type] for our specific project. Include only decisions we made. No N/A or TBD sections."

---

### MULTI-AGENT SQUAD PROMPTS

**Typical Output:** Sequential handoffs between agents; composite output (options + analysis + ADRs).

#### Hallucination Risk Profile

| Risk | Signal | Fix |
|------|--------|-----|
| **Hallucination Cascade** | Agent 2 builds on Agent 1's [INFERRED] assumption without questioning | Missing validation gates. Rerun: "Agent 2 MUST validate Agent 1's flagged items before proceeding." |
| **Lost Context** | Agent 3's output contradicts Agent 1's findings | Agents didn't read prior handoff. Rerun: "Provide full prior agent output; require Agent 3 to confirm agreement." |
| **Confidence Drift** | Agent 1 outputs "Confidence: 3/5". Agent 3 outputs "Confidence: 5/5" for same claim | No confidence tracking. Rerun: "Each agent must preserve and justify confidence scores." |
| **Inconsistent Terminology** | Agent 1 calls it "option A"; Agent 3 calls it "approach 1" | No terminology mapping. Rerun: "Use consistent naming throughout handoff." |

#### Quality Rubric

- [ ] **Validation Gates**: Each agent validates prior agent's flagged items ([INFERRED], [SPECULATIVE], low confidence)
- [ ] **Confidence Tracking**: Confidence scores preserved through handoff; updated if agent challenges
- [ ] **Assumption Register**: Each agent documents assumptions made; passed to next agent
- [ ] **Escalation Rules**: If confidence < 3, agent escalates rather than proceeding
- [ ] **Terminolog Consistency**: Definitions preserved through handoff (no rename mid-pipeline)
- [ ] **Traceability Chain**: Final output traces back to initial input through all agent handoffs
- [ ] **No Silent Disagreement**: If agents disagree, conflict is documented (not papered over)
- [ ] **Explicit Integration**: Final agent explicitly reconciles differences, doesn't merge naively

#### Acceptance Threshold

**PASS:** 6–8 items checked  
**FAIL:** 2+ items unchecked; rerun with: "Add validation gates. Each agent must confirm prior assumptions. Mark assumptions and confidence scores. Escalate low confidence."

---

## Guardrails & Evidence Checks (Advanced)

Use these checks if you have time and output will drive a major decision.

### 1. Invented Constraints

**Signal:** Output assumes constraints you never provided.

**Check Method:**
1. List every constraint mentioned in the output
2. For each constraint, verify: Did I state this? (Yes/No)
3. If "No," mark as [INFERRED] or [INVENTED]

**Example:**
```
Output mentions: "Monolith works for teams <10 engineers"
Your input: "We have 5 engineers" (stated team size)
Assessment: Constraint is INFERRED (from best practice, not from input)

Output mentions: "API must use GraphQL for real-time subscriptions"
Your input: No mention of real-time subscriptions
Assessment: Constraint is INVENTED (hallucination)
```

**Action:** Rerun with: "Do not invent constraints. If information is missing, output [INPUT MISSING]."

---

### 2. Buzzword Density

**Signal:** Output uses trendy words without definition.

**Check Method:**
1. Scan for buzzwords: "microservices," "cloud-native," "serverless," "event-driven," "AI-powered"
2. For each buzzword, ask: Did the output explain why THIS buzzword fits?
3. Generic praise ("microservices are scalable") doesn't count as explanation

**Example:**
```
Output: "Use a cloud-native microservices architecture for resilience"
Explanation: None (generic praise)
Assessment: ✗ BUZZWORD (no grounding)

Output: "Use microservices because your 10M-record dataset requires isolation per domain, 
         and you have team expertise in service-oriented testing"
Explanation: Explicit (references your constraints)
Assessment: ✓ GROUNDED
```

**Action:** Rerun with: "Explain why each recommendation fits YOUR specific constraints. Avoid generic praise."

---

### 3. Confidence Without Evidence

**Signal:** Output rates things highly without stating basis.

**Check Method:**
1. Find all 1–5 ratings in the output
2. For each rating, ask: What's the basis? (Stated constraint? Market data? Assumption?)
3. "Obviously" or "clearly" without context = no evidence

**Example:**
```
Output: "This architecture scores 4/5 on cost-effectiveness"
Evidence: None stated
Assessment: ✗ NO BASIS (hallucination)

Output: "This architecture scores 4/5 on cost-effectiveness. BASIS: AWS pricing for RDS + EC2 
         at 1M request/month = $2,800/month, fits your $50K/month budget with 94% cushion"
Evidence: Explicit market data + constraint reference
Assessment: ✓ GROUNDED
```

**Action:** Rerun with: "Ground every rating in explicit evidence. If you're guessing, output [UNKNOWN]."

---

### 4. False Equivalence

**Signal:** Output treats incomparable things as equal.

**Check Method:**
1. Look for comparisons (e.g., "Option A vs. Option B")
2. Ask: Are these actually comparable? Do they solve the same problem?
3. False equivalence: Comparing apples (deployment model) to oranges (communication style)

**Example:**
```
Output: "Comparison: Monolith (simple) vs. Microservices (complex)"
Assessment: ✗ FALSE EQUIVALENCE (simplicity depends on YOUR use case; 
                                  monolith can be complex too)

Output: "Comparison: Monolith (good for <20 engineers, higher latency) 
                    vs. Microservices (good for >50 engineers, lower latency)"
Assessment: ✓ COMPARABLE (trade-offs explicitly tied to your context)
```

**Action:** Rerun with: "Only compare architectures that solve the same problem. State trade-offs clearly."

---

### 5. Unverified Examples

**Signal:** Output includes examples that might be invented.

**Check Method:**
1. If output references "companies like Uber," "typical SaaS," or "real-world examples," verify
2. Can you look this up? Is the example real and accurately described?
3. Vague examples ("imagine a social network") are often invented

**Example:**
```
Output: "Stripe uses event-driven architecture for payment processing"
Verification: Partially true (Stripe does use event streaming; they also use other patterns)
Assessment: ✗ OVERSIMPLIFIED (not wrong, but incomplete)

Output: "Netflix uses microservices because they need <100ms latency for video delivery"
Verification: Netflix does use microservices; true latency driver
Assessment: ✓ ACCURATE
```

**Action:** Rerun with: "Use only documented examples. If using examples, state your source."

---

### 6. Escaping the Frame

**Signal:** Output ignores constraints and proposes something out of scope.

**Check Method:**
1. Review your explicit constraints
2. Look for output that contradicts them
3. Example: You said "no on-prem," output says "data center deployment"

**Example:**
```
Your constraints: Budget $50K/month, team size 5 engineers, AWS only
Output recommends: Kubernetes on self-managed on-prem clusters
Assessment: ✗ OUT OF FRAME (requires infrastructure team, violates budget)

Your constraints: "Must use Python for backend"
Output recommends: Rust microservices
Assessment: ✗ OUT OF FRAME (violates technology constraint)
```

**Action:** Rerun with emphasis: "Your constraints are: [restate them]. Reject any recommendation that violates them."

---

### 7. Asymmetric Information

**Signal:** Output presents imbalanced information (heavily favors one option).

**Check Method:**
1. For each architecture option, count pros and cons
2. Look for uneven treatment (Option A: 5 pros, 1 con; Option B: 1 pro, 5 cons)
3. Legitimate options should have comparable trade-offs

**Example:**
```
Option A (Monolith): Simple, fast to market, easy testing (3 pros)
Option B (Microservices): Distributed, hard to debug, higher cost, operational overhead (3 cons, 0 pros)
Assessment: ✗ IMBALANCED (Option B presented as all downside)

Option A (Monolith): Simple (pro), scales poorly (con), easy testing (pro), hard to scale team (con)
Option B (Microservices): Scales well (pro), complex operations (con), team independence (pro), cost overhead (con)
Assessment: ✓ BALANCED
```

**Action:** Rerun with: "Present each option fairly. For each option, state 2–3 pros AND 2–3 cons."

---

## Meta-Check: Am I Over-Relying on AI?

Before acting on any output, ask:

- [ ] Have I (a human expert) reviewed this output?
- [ ] Does this align with my domain knowledge?
- [ ] Would I make the same recommendation without AI?
- [ ] What's my confidence in this output? (> 70%? > 90%?)
- [ ] What's the cost of being wrong?

**Red Flags:**
- You're acting on output you don't fully understand
- Output contradicts your domain knowledge, but you're proceeding anyway
- You haven't had a subject-matter expert review it
- You're unable to explain the recommendation to your team

**Green Flags:**
- You understand the reasoning, agree with the approach
- Output aligns with your knowledge + adds new insights
- You've verified key facts (costs, capabilities, constraints)
- You could explain this recommendation to your executive sponsor
- You have a fallback plan if this recommendation is wrong

---

## Rubric Summary Table

| Prompt Type | Hallucination Risk | Key Checks | Threshold |
|-------------|-------------------|-----------|-----------|
| **Requirements Clarification** | Invented requirements, over-interpretation | Ambiguities grounded, NFRs marked [INFERRED], stakeholder KPIs specific | 5/6 |
| **Architecture Exploration** | Generic options, meaningless ratings, invented costs | Constraint-driven filtering, grounded ratings, cost transparency | 6/8 |
| **Adversarial Review** | Invented risks, generic critiques, unactionable recommendations | Specific failures, domain-aware, actionable, evidence-based | 6/8 |
| **Documentation (ADRs, etc.)** | Invented options/consequences, template dependency failure, pseudo-documentation | Template compliance, options grounded, consequences realistic, no N/A sections | 7/9 |
| **Multi-Agent Squad** | Hallucination cascade, lost context, confidence drift | Validation gates, confidence tracking, assumption register, escalation rules | 6/8 |

---

## Workflow: When to Use This Rubric

### Scenario 1: Quick Validation (You're in a Hurry)
1. Run **Fast Checks** section (3 minutes)
2. If output passes all 3, proceed with caution
3. If output fails any check, rerun with clarification

### Scenario 2: Major Decision (You Have Time)
1. Run **Fast Checks** (3 minutes)
2. Run **Section-Specific Rubric** for your prompt type (10–15 minutes)
3. If passes threshold, proceed; if not, rerun with feedback
4. Optional: **Advanced Guardrails** if output will drive significant change

### Scenario 3: Skeptical Review (You Distrust the Output)
1. Run all **Advanced Guardrails Checks** (15–20 minutes)
2. For each failed check, note specific AI behavior
3. Rerun prompt with feedback tailored to each failure
4. Iterate until output passes checks

---

## When This Rubric Is Most Valuable

- First time using a library prompt (learn its patterns)
- Output will influence a major architectural decision
- Team is unfamiliar with AI-assisted architecture
- You're building an unfamiliar domain (new risks)
- Attendee will act on output without expert review

---

## When This Rubric May Be Overkill

- Brainstorming/ideation (where uncertainty is the point)
- Low-stakes output (you'll validate with experts anyway)
- You're running multiple iterations (fast feedback loop already)

---

## Appendix: Common AI Failure Patterns in Architecture Prompts

| Failure | Root Cause | Detection | Prevention |
|---------|-----------|-----------|------------|
| **Invented constraint that inflates scope** | "All X" prompts encourage AI to infer | Output mentions constraints you didn't provide | Explicitly: "Do not invent constraints. Output [INPUT MISSING] if needed." |
| **Recommendation that contradicts your budget** | AI doesn't validate cost against constraint | Cost estimate breaks your budget ceiling | Require: "Every cost estimate must reference explicit budget constraint." |
| **Option that requires expertise you don't have** | Generic options assume mid-size teams | Monolith recommended for 2-person startup | Add: "Option must fit our team size & skill level explicitly." |
| **Hallucinated best practice** | "Best practice" is jargon for "I've seen this pattern" | Output cites best practice without grounding | Demand: "Ground every 'best practice' in constraint or evidence." |
| **Tradeoff ratings that don't match reality** | Ungrounded 1–5 scale | Monolith rated high on scalability for all scenarios | Require: "Rate only against explicit constraints. Output [UNKNOWN] if missing." |
| **Failure mode that isn't specific** | "Generic risks apply to every architecture" | Output: "Database could fail" (applies to all databases) | Demand: "Specific trigger? How do you detect it? How long to recover?" |

---

## Questions? Next Steps

**After using this rubric:**
- What checks were most useful?
- What checks revealed problems?
- What did AI do well that you didn't expect?
- Where is AI weakest in your domain?

**To improve prompt quality over time:**
1. Document which checks fail most often
2. Add those as standard guardrails to your customized prompt
3. Share learnings with your team
4. Iterate on the playbook

**Feedback?** This rubric is designed to improve. If you find failure patterns not listed here, document them and share with your team.
