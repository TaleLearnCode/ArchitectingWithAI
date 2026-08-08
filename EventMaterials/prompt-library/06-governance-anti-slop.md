# 06 — Governance & Anti-Slop Prompts

*Enforce quality guardrails to keep AI-generated output grounded, specific, and actionable.*

---

## Section Purpose

AI is very good at generating plausible-sounding outputs that are actually hallucinations: invented capabilities, generic filler, confident claims without evidence, repeated content, buzzwords instead of specifics.

These prompts help you:
- **Detect hallucinations in your own output** — confident but ungrounded claims, invented technologies, made-up measures
- **Enforce document boundaries** — keep ADRs focused on architecture, not implementation; keep requirements focused on needs, not solutions
- **Remove generic filler** — cut redundancy, replace vague statements with specifics, flag unsubstantiated claims
- **Validate constraint grounding** — ensure recommendations actually respect your constraints, not just claim to
- **Audit AI workflow quality** — check that earlier sections (1–5) didn't introduce hallucinations

**Thesis Connection:** AI can generate slop. Architects supply judgment by catching it. These prompts teach you what to look for and how to automate detection.

---

## Prompt Catalog

---

## ✓ Anti-Slop Content Review

### Use This When
You have received generated text (ADR, specification, process flow, etc.) and need to catch generic filler, ungrounded claims, and repeated content before sharing it with stakeholders.

### When NOT to Use
Do not use on hand-written documentation; this is for catching AI output issues only.

### Prerequisites
- Generated text (from AI)
- Context about the project (constraints, requirements, domain)
- Familiarity with your organization's quality bar for documentation

**Validation Rule:** If you don't know what "slop" looks like in your domain, run this prompt and use output to calibrate your eye.

---

### Input Example

```
GENERATED TEXT (ADR excerpt):
"We chose microservices because they are more scalable and easier to maintain. 
They provide better performance. Microservices are the modern approach to architecture.
They scale better than monoliths. Using microservices ensures we can handle growth."

CONTEXT:
Constraints: 3-person team, 6-month launch window, $150K annual budget
Requirements: 100K concurrent users, 99.5% availability
Domain: E-commerce platform
```

---

### The Prompt

```text
Review this generated text and identify instances of slop (generic filler, ungrounded claims, repeated content).

For each instance of slop, provide:
1. The problematic phrase
2. Why it's slop (generic? repeated? ungrounded? out-of-scope?)
3. What specific claim it should make instead

SLOP PATTERNS TO CATCH:

Generic adjectives without metrics:
❌ "more scalable", "better performance", "easier to maintain", "more reliable"
✓ "scales from 10k to 100k concurrent users"
✓ "reduces deployment time from 2 hours to 15 minutes"
✓ "reduces operational overhead by eliminating need for DBAs"

Confident claims without basis:
❌ "Microservices are the best approach"
❌ "This is a proven solution"
❌ "Cloud-native architecture ensures reliability"
✓ "This approach handles our 100k concurrent user load based on [specific reason]"
✓ "This has been validated on similar projects with [specific similarity]"

Invented technologies or capabilities:
❌ "AI-powered auto-scaling"
❌ "Self-healing infrastructure"
❌ "Zero-downtime deployment" (without specifying how)
✓ Name specific tools: "AWS RDS with Multi-AZ failover", "Kubernetes with circuit breakers"

Repeated content:
❌ Same point made twice in different words
✓ Remove repetition; keep unique claims only

Repeated assumptions:
❌ "Team skill in Kubernetes" mentioned 3 times
✓ Mention once, reference as needed

Out-of-scope details:
❌ Implementation details in an ADR (e.g., "Use Node.js clustering module")
❌ Business justification in a technical specification
✓ Keep implementation in impl docs; business in business case docs

Ungrounded confidence:
❌ "Guaranteed 99.99% uptime"
❌ "This eliminates all scaling concerns"
✓ "Supports up to 1M requests/sec, which exceeds our 500k/sec peak projection by 2x"

Format your output as a CSV or table:
Slop Type | Problematic Phrase | Why It's Slop | Suggested Rewrite

***
[PASTE YOUR GENERATED TEXT HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Strictness** | You want zero tolerance | Add: "Flag even slightly vague claims; err on the side of caution" |
| **Leniency** | You want quick feedback | Add: "Focus on egregious slop only; skip minor imprecision" |
| **Domain focus** | You care about specific patterns | Add: "Focus on generic tech claims; skip vague business language" |
| **Rewrite suggestions** | You want fixes, not just flagging | Add: "For each flagged phrase, suggest a specific, grounded rewrite" |

---

### Validation Checklist

- [ ] **Patterns consistent:** All flagged phrases match stated slop patterns (not subjective opinions)
- [ ] **Specificity:** Each flagged phrase is concrete (not "writing could be better")
- [ ] **Rewrites actionable:** Suggested rewrites are specific and measurable (not more vague)
- [ ] **False positives minimized:** Judgment calls explained (e.g., "This is borderline generic")

---

## ✓ Constraint Grounding Validation

### Use This When
You want to verify that generated recommendations actually respect your stated constraints (budget, team size, deployment model, regulatory requirements). Catch hallucinations where AI proposes solutions that contradict your constraints.

### When NOT to Use
Skip if your prompts already include explicit constraint validation gates.

### Prerequisites
- Generated recommendation (architecture, technology choice, design decision)
- Explicit constraints (written down and shared with AI)
- Authority to ask stakeholders "Is this constraint real?" (because AI might question it)

**Validation Rule:** If constraint is ambiguous, output will flag it. Use that as signal to clarify the constraint.

---

### Input Example

```
STATED CONSTRAINTS:
- Budget: $150K/year (all infrastructure, vendors, tooling)
- Team: 3 backend engineers (no specialized ops/DBAs)
- Deployment: AWS, self-managed (no Lambda/managed services)
- Compliance: No external APIs (customers' requirement; closed network)
- Timeline: 6 months to production
- Operational maturity: Low (< 2 years running production systems)

GENERATED RECOMMENDATION:
"We recommend a microservices architecture using Kubernetes. Deploy to multi-region AWS with 
active-active failover. Use Kafka for event streaming. Implement distributed tracing with Jaeger.
This provides high availability and scales to millions of requests."
```

---

### The Prompt

```text
Validate that this recommendation respects ALL stated constraints.

For each constraint, answer:
1. Is the recommendation compatible with this constraint? (Yes/No/Unknown)
2. If No: What about the recommendation conflicts? (Be specific)
3. If Unknown: What clarification is needed?

Use this format:

CONSTRAINT: [State constraint]
STATUS: [COMPATIBLE / VIOLATED / UNKNOWN]
EVIDENCE: [How recommendation aligns or conflicts with constraint]
ACTION: [Proceed / Clarify / Revise recommendation]

Example:
CONSTRAINT: "No specialized ops team; team of 3 backend engineers"
STATUS: VIOLATED
EVIDENCE: "Kubernetes requires 24/7 monitoring, security patching, version upgrades. 
3 engineers cannot maintain this without hiring additional DevOps staff."
ACTION: Revise recommendation to use AWS ECS with managed container orchestration OR 
hire DevOps specialist (costs money, violates budget constraint)

RED FLAGS (Hallucination indicators):
❌ "This constraint is outdated; modern teams always use [technology]"
   (Dismissing constraint; likely hallucination)
❌ "You don't need to worry about this constraint"
   (Minimizing constraint; likely hallucination)
❌ "This constraint will be fine once you [major investment]"
   (Requiring new investment not budgeted)

If you find violations, output them explicitly. Do not proceed silently.

***
[PASTE YOUR CONSTRAINTS AND RECOMMENDATION HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Strictness** | Zero tolerance for violations | Add: "If ANY constraint is violated, output 'REVISION REQUIRED'" |
| **Severity-based** | Some constraints matter more | Add: "Mark constraints as HARD (non-negotiable) or SOFT (flexible); flag HARD violations only" |
| **Cost focus** | Budget is your main concern | Add: "For each recommendation, estimate annual cost (infrastructure, team headcount, vendor fees)" |
| **Compliance focus** | Regulatory constraints critical | Add: "Prioritize regulatory/compliance constraint violations over others" |

---

### Validation Checklist

- [ ] **All constraints reviewed:** Every stated constraint examined (not skipped)
- [ ] **Violations explicit:** Any conflicts with constraints stated clearly with examples
- [ ] **No dismissal:** Constraints not minimized or reframed ("actually this constraint isn't needed")
- [ ] **Cost estimated:** If budget constraint, recommendation includes cost breakdown
- [ ] **Clarifications concrete:** If "Unknown," specific question asked (not vague)

---

## ✓ Hallucination Detection Checklist

### Use This When
You want to catch invented technologies, made-up measures, false confidence, and other hallucinations before they enter your decision-making.

### When NOT to Use
Do not use on architectural decisions that have already been validated by humans.

### Prerequisites
- Generated text with architecture choices, technology recommendations, or capability claims
- Knowledge of what technologies actually exist and what your team can actually do
- Willingness to ask "Are you sure?" when AI sounds confident

**Validation Rule:** When in doubt, look it up. Hallucinations sound confident but can't be validated.

---

### Input Example

```
GENERATED TEXT:
"Use serverless functions with auto-scaling to handle catalog uploads. 
The system will auto-correct data inconsistencies using AI. 
We recommend AWS AppSync for real-time synchronization. 
This provides guaranteed 99.99% uptime and eliminates operational overhead."
```

---

### The Prompt

```text
Review this text and flag hallucinations: invented tech, made-up capabilities, false guarantees, 
unvalidated claims.

RED FLAGS TO CHECK:

Invented or misnamed technologies:
❌ "AI-powered data correction" (What actual technology? Doesn't exist)
❌ "Quantum-accelerated search" (Not a real product you can use)
✓ "Machine learning model to detect data anomalies using SageMaker" (specific, real product)

Capabilities that don't exist:
❌ "Self-healing infrastructure that detects and fixes problems automatically"
❌ "Zero-downtime deployment with zero data loss"
✓ "Blue-green deployment with ~5-minute switchover window"

Guaranteed outcomes without backing:
❌ "Guaranteed 99.99% uptime"
❌ "Eliminates operational overhead"
❌ "Solves all scaling concerns"
✓ "Supports horizontal scaling up to 1M requests/second based on [vendor documentation]"

Vendor claims without validation:
❌ "AWS RDS can handle 1 billion transactions per second" (Sounds plausible but unverified)
❌ "Kubernetes automatically optimizes costs" (It doesn't)
✓ "RDS supports horizontal scaling via read replicas (up to 5 per instance, per AWS docs)"

Unvalidated assumptions:
❌ "Your team will easily learn Kubernetes in 2 weeks"
❌ "This architecture is simpler than microservices"
✓ "Monolith requires less operational complexity than Kubernetes (based on team having no K8s experience)"

For each flagged item, provide:
1. The problematic claim
2. Why it's hallucination (invented? unvalidated? overgeneralized?)
3. What you'd need to verify it (documentation link? vendor spec? test?)
4. Safer alternative (if one exists)

Format:
Hallucination | Type | Why | Verification Needed | Safer Alternative

Do NOT assume any technology or capability is real unless explicitly validated.

***
[PASTE YOUR GENERATED TEXT HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Tech-focused** | You want to catch invented technologies | Add: "Check every vendor name and product name; flag if you can't confirm it exists" |
| **Capability-focused** | You want to catch impossible claims | Add: "Flag any claim of zero cost, zero complexity, or guaranteed outcomes" |
| **Assumption-focused** | You want to catch hidden assumptions | Add: "For each claim, state the assumption required to make it true" |
| **Expert mode** | You have domain expertise | Add: "You are an expert in [domain]. Flag anything that contradicts domain best practices" |

---

### Validation Checklist

- [ ] **Technology names verified:** Every technology mentioned is a real, existing product
- [ ] **Capabilities grounded:** Claims about what systems can do are traceable to documentation or specs
- [ ] **Assumptions exposed:** Any assumption required for a claim to be true is stated
- [ ] **False confidence flagged:** Confident-sounding but ungrounded claims marked
- [ ] **Alternatives offered:** For flagged hallucinations, safer alternatives proposed

---

## ✓ Document Boundary Enforcement

### Use This When
You want to ensure content stays in scope. ADRs shouldn't contain implementation details. Requirements shouldn't prescribe solutions. Process flows shouldn't specify technology.

### When NOT to Use
Skip if your documents are already well-scoped or reviewed by subject matter experts.

### Prerequisites
- Document type clear (ADR, specification, requirements, process flow, risk register, etc.)
- Document scope definition (what belongs, what doesn't)
- Tolerance for boundary violations (strict or flexible?)

**Validation Rule:** If you're unsure what belongs in a document type, ask this prompt first.

---

### Input Example

```
DOCUMENT TYPE: Architecture Decision Record (ADR)

CONTENT (excerpt):
"We chose PostgreSQL because it's reliable and open source. 
We'll implement sharding using a custom Python script.
The script reads from a configuration file in YAML format.
Developers should install dependencies using pip install -r requirements.txt.
Monitoring is configured in our Grafana dashboards..."

SCOPE DEFINITION:
ADRs should contain:
- The decision and context
- Options considered
- Tradeoffs
- Consequences

ADRs should NOT contain:
- Implementation details (how to code it)
- Deployment procedures (how to set it up)
- Monitoring dashboards (operational details)
```

---

### The Prompt

```text
Review this [DOCUMENT TYPE] and flag content that violates document scope.

REFERENCE SCOPE:
[Your scope definition: what belongs in this document type, what doesn't]

For each out-of-scope item, identify:
1. The problematic content
2. What scope it violates (what category does it belong to?)
3. Where it should go instead
4. Why it matters (having it in wrong place makes both documents weaker)

Format:
Out-of-Scope Content | Violates | Belongs In | Why It Matters

Red flags by document type:

ADRs (Architecture Decision Records):
❌ Step-by-step implementation instructions
❌ Specific code examples or configuration
❌ Operational procedures (how to deploy, monitor, troubleshoot)
✓ Decision rationale, options considered, tradeoffs, consequences

Requirements:
❌ Specific technology recommendations ("Use Kubernetes")
❌ Implementation constraints ("No loops in this function")
❌ Design patterns ("Use the Repository pattern")
✓ Business needs, functional behavior, NFR targets, constraints

Process Flows:
❌ Technology prescriptions ("Query database with SQL SELECT")
❌ Specific tools ("Use Slack API to send notification")
❌ Code snippets
✓ Steps, actors, decision points, timing, error handling

Specifications:
❌ "Why we chose this approach" (belongs in decision records)
❌ "How to deploy this" (belongs in runbooks)
✓ "What this component does", interfaces, contract, behavior

Risk Register:
❌ Mitigation action plans (belongs in risk response plan)
❌ Root cause analysis of past incidents (belongs in postmortems)
✓ Risk triggers, probability, impact, monitoring, escalation

For each flagged item, explain why it weakens the document and where it belongs.

***
[PASTE YOUR DOCUMENT CONTENT HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Strict** | You enforce boundaries rigidly | Add: "Flag any content that stretches beyond the scope, even slightly" |
| **Lenient** | Boundaries are flexible | Add: "Only flag egregious scope violations; some overlap is OK" |
| **Cross-reference** | You want to know where content belongs | Add: "For each out-of-scope item, specify exactly which document it should be in" |
| **Educator mode** | You want to teach scope | Add: "Explain why each scope boundary matters for document clarity and reusability" |

---

### Validation Checklist

- [ ] **All violations identified:** Every out-of-scope section flagged (not missed)
- [ ] **Specificity:** Violations reference exact content (phrases or sections), not vague "this is out of scope"
- [ ] **Clarification provided:** For each violation, clear explanation of why it violates scope
- [ ] **Destination clear:** For each violation, specific document type or section identified where it belongs
- [ ] **No false positives:** Flagged content actually violates boundaries (not edge cases or judgment calls)

---

## ✓ Assumption Audit

### Use This When
You want to find unmarked inferences in completed documentation. Content presented as fact but actually inferred from context or assumptions.

### When NOT to Use
Do not use if your team never references assumptions (documentation debt incoming).

### Prerequisites
- Completed documentation (ADRs, specifications, architecture docs, assumptions register if available)
- Project context (constraints, requirements, team profile, decisions made)
- Familiarity with what was explicitly stated vs. what is implied

**Validation Rule:** "Inferred" is not bad; "unmarked inferred" is. This prompt surfaces unmarked inferences.

---

### Input Example

```
DOCUMENTATION (Excerpt from ADR):
"We've chosen a monolithic architecture. The team can maintain this effectively. 
This approach will support our scaling needs for the next 2 years. 
Database queries respond in milliseconds. The system is highly available."

PROJECT CONTEXT (What was actually stated):
- Team: 3 backend engineers
- Constraints: No Kubernetes, no distributed system ops experience
- SLA: 99.5% availability
- Growth projection: 10x user load possible within 2 years
```

---

### The Prompt

```text
Review this documentation and flag unmarked inferences (content stated as fact but actually inferred).

For each inferred claim, identify:
1. The statement that is inferred (exact quote)
2. What was explicitly stated vs. what was inferred
3. Why the inference is reasonable (what basis supports it?)
4. What evidence would validate or refute it?
5. How to mark it properly

Format:
Statement | Explicitly Stated | Inferred From | Basis | Validation | Proper Marking

Examples of unmarked inferences:

❌ "The team can maintain this effectively" (Unmarked inference)
   - Explicit: Team of 3 engineers, no Kubernetes experience
   - Inferred: They can maintain a monolith
   - Basis: Monoliths require less ops complexity than distributed systems
   - Proper marking: "[INFERRED] The team can maintain a monolith effectively (based on 3-person team size and no distributed ops experience)"

❌ "Database queries respond in milliseconds" (Unmarked inference)
   - Explicit: No numbers given
   - Inferred: Millisecond performance is achievable
   - Basis: Unstated assumptions about query optimization, indexing strategy, query volume
   - Proper marking: "[INFERRED] Database queries will respond in milliseconds (assumes optimized queries, appropriate indices, and load < 1000 QPS)"

❌ "System is highly available" (Unmarked inference)
   - Explicit: 99.5% SLA stated
   - Inferred: This is "highly available"
   - Basis: 99.5% is business standard but not "highly available" (99.99% is industry standard for "highly available")
   - Proper marking: "[INFERRED] System meets 99.5% SLA, which is business-standard availability (not 'highly available' in industry terms of 99.99%+)"

RED FLAGS (Inference patterns):

Unstated metrics or numbers:
❌ "Fast performance", "scales well", "reliable"
   These are inferred without quantification

Unstated audience or context:
❌ "Appropriate for our use case"
   Assumes "our use case" is clear; may not be to someone else

Unstated technology assumptions:
❌ "Database handles this load"
   Assumes you've tested or verified load capacity

Unstated team or organizational context:
❌ "Easy to understand"
   Assumes reader has same technical background

For each inferred statement, recommend how to mark it clearly so readers understand it's not fact.

Use these markers:
[INFERRED] — Conclusion drawn from stated facts
[ASSUMED] — Explicit assumption required for claim to be true
[REQUIRES VALIDATION] — Needs testing or stakeholder confirmation
[HYPOTHESIS] — Educated guess, needs evidence

***
[PASTE YOUR DOCUMENTATION HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Audit mode** | You're reviewing completed docs | Add: "Assume this documentation is complete; flag any inferences not marked" |
| **Collaborative mode** | You're helping author improve | Add: "Suggest how to rewrite unmarked inferences to be clear and marked" |
| **Risk focus** | You care about high-impact inferences | Add: "Flag only inferences that affect major decisions or commitments" |
| **Pedagogical** | You're teaching the team | Add: "For each unmarked inference, explain why marking it helps readers" |

---

### Validation Checklist

- [ ] **Comprehensive:** All unmarked inferences identified (not just obvious ones)
- [ ] **Accuracy:** Each flagged item is genuinely an inference (not a direct statement)
- [ ] **Basis explained:** For each inference, reasoning shown for why it's inferred
- [ ] **Validation path clear:** For each inference, path to validate it described
- [ ] **Helpful marking:** Suggested marking is clear and useful to readers

---

## ✓ Confidence Scoring

### Use This When
You want to rate your own (or AI's) confidence in specific claims, especially on high-impact decisions. Makes clear what's certain vs. speculative.

### When NOT to Use
Skip if your organization does not track decision confidence or uses subjective judgment.

### Prerequisites
- Text with claims that affect decisions (architecture recommendation, risk assessment, capability projection)
- Basis for confidence judgment (data, experience, prior validation, expert opinion)
- Willingness to say "I don't know" when confidence is low

**Validation Rule:** Confidence should be grounded in evidence, not in how well the text is written.

---

### Input Example

```
CLAIMS TO SCORE:
1. "Team of 3 engineers can maintain a microservices architecture"
2. "Serverless functions will reduce ops overhead by 60%"
3. "System will handle 100K concurrent users"
4. "Compliance team has approved this approach"
5. "Marketplace features can be added post-launch without refactoring"
```

---

### The Prompt

```text
For each claim, rate confidence 1–5 and justify the rating.

Use this scale:
5 = Certain (validated, tested, past precedent, stakeholder confirmed)
4 = High confidence (strong evidence, expert consensus, proven pattern)
3 = Medium confidence (educated guess, reasonable assumptions, some validation)
2 = Low confidence (speculative, high assumption, limited evidence)
1 = Very low confidence (wild guess, lots of unknowns, contradicts experience)

For each claim, provide:
1. Confidence score (1–5)
2. Basis for the score (what evidence supports this confidence level?)
3. What would increase confidence? (What would need to be true to move from 3 → 4?)
4. What would decrease confidence? (What could go wrong?)
5. Action if confidence < 3 (research? spike? assumption documentation?)

Format:
Claim | Score | Evidence | Increase Confidence | Decrease Confidence | Action if Low

Example:
"Team can maintain microservices" | 2 | 
Evidence: "Team has 0 experience with distributed systems; 2 engineers have ops background but monolith-focused"
Increase: "Run 2-week Kubernetes training; maintain one microservice in staging for 1 month"
Risk: "Any engineer leaves; team overwhelmed by service mesh complexity; inter-service debugging takes 3x longer than expected"
Action: "SPIKE: Kubernetes POC with actual team. If score stays <3, switch to monolith"

RED FLAGS (Low-confidence indicators):

Unvalidated assumptions:
1. "Team will adapt quickly" (Assumption; no basis)
2. "Scaling will be linear" (Assumption; depends on architecture)
3. "Customers won't mind eventual consistency" (Assumption; needs validation)

Untested claims:
1. "System handles 100K users" (Untested; maybe it does, maybe it doesn't)
2. "Compliance is approved" (Untested; have you asked?)
3. "API response time is < 100ms" (Untested; need benchmarks)

Contradicted by evidence:
1. "Team easily learns new tech" (But past projects show slow adoption)
2. "Architecture is simple" (But documentation is 50 pages)

For any claim with score < 3, specify:
- Investigation needed (what would increase confidence?)
- Timeline (when do you need to know?)
- Risk if wrong (what happens if assumption is invalid?)

***
[PASTE YOUR CLAIMS HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Risk mode** | You're managing high-stakes decisions | Add: "For low-confidence claims, flag as REQUIRES VALIDATION before proceeding" |
| **Team calibration** | You're teaching the team | Add: "Include examples of how past projects calibrated confidence incorrectly" |
| **Prioritization** | You have many claims to score | Add: "Rank claims by business impact; focus on high-impact, low-confidence items first" |
| **Research planning** | You need to know what to investigate | Add: "For each claim with score < 3, suggest specific research to increase confidence" |

---

### Validation Checklist

- [ ] **Scores grounded:** Each score references specific evidence (not just intuition)
- [ ] **Calibration realistic:** Scores account for unknowns (not overconfident)
- [ ] **Paths to improvement clear:** For each low-confidence claim, specific investigation path stated
- [ ] **Risks identified:** For each low-confidence claim, "what goes wrong?" articulated
- [ ] **Team alignment:** Scores reflect team expertise and experience (not external benchmark)

---

---

## How to Use

### Post-Generation Review (After Sections 1–5)

1. **Run Anti-Slop Content Review** on every generated document
   - Input: Generated text (ADR, process flow, specifications, etc.)
   - Output: Flagged phrases + suggested rewrites
   - Example flagged claims: "more scalable", "easier to maintain", "better performance"
   - Rewrite to specifics: "scales to 100k concurrent users (vs. current 10k)" or "cuts TTM by 3 weeks"

2. **Run Constraint Grounding Validation**
   - Input: Generated architecture recommendation + stated constraints
   - Output: Verification that recommendation respects each constraint (or flagged violations)
   - Example: "You said 'no external APIs' but proposed Stripe payment processing" → halt

3. **Run Hallucination Detection Checklist**
   - Input: Generated architecture, technology choices, capability claims
   - Output: Flagged invented capabilities or ungrounded claims
   - Example red flags: "AI-powered auto-scaling" (without detail), "guaranteed 99.99% uptime" (without backing assumptions)

4. **Run Document Boundary Enforcement**
   - Input: Architecture specification or ADR
   - Output: Identify content that violates document boundaries
   - Example: ADR discussing implementation details ("use Node.js clustering module") should move to implementation docs

### Pre-Review Gate (Before Team Review)

Run all 4 reviews above on generated output. If any review flags >3 issues, have AI revise or mark for human refinement.

### Audit Mode (Quality Assurance on Prior Work)

1. **Assumption Audit** on completed architecture documentation
   - Input: All ADRs, specifications, process flows
   - Output: List of unmarked inferences (claims presented as fact but actually inferred)
   - Example: "Assumes team has Kubernetes experience" (not stated explicitly)

2. **Confidence Scoring** on risky decisions
   - Input: ADRs or architecture choices with high business impact
   - Output: Each claim rated (high/medium/low confidence) with justification
   - Example: "High confidence: 3-person team can maintain monolith (based on hiring plan). Medium confidence: async messaging will reduce latency (depends on queue implementation)."

---

## Customization Guidance

### By Document Type

- **ADRs:** Focus on constraint grounding + assumption audit (ADRs are decision *justifications*, not fact statements)
- **Process Flows:** Focus on document boundary enforcement (flows shouldn't prescribe technology) + hallucination detection (invented steps)
- **Specifications:** Focus on hallucination detection + boundary enforcement (specs are *what*, not *how*)
- **Risk Registers:** Focus on confidence scoring (probability/impact estimates are often too confident or too pessimistic)

### By Review Thoroughness

- **Quick scan (15 minutes):** Anti-slop review + hallucination checklist only
- **Standard review (30 minutes):** All 4 post-generation reviews above
- **Comprehensive (1 hour):** Standard review + assumption audit + confidence scoring + stakeholder spot-check

### By Organizational Context

- **Startup (speed > rigor):** Anti-slop review mandatory; skip assumption audit until architecture stabilizes
- **Mid-scale (balancing speed + credibility):** Standard review mandatory; spot-check confidence scoring on high-impact decisions
- **Enterprise (rigor > speed):** All reviews mandatory; audit mode on all completed architecture

---

## Quality Checks

### Success Indicators for Anti-Slop Review

- ✅ Generic claims ("this is scalable") are replaced with specifics (scales to X concurrent users, X GB data, X transactions/sec)
- ✅ Invented capabilities are caught (no "AI-powered" magic, no assumed vendor features without validation)
- ✅ Out-of-scope content is identified (implementation details in ADRs, solutions in requirements, etc.)
- ✅ Assumptions are explicit (marked `[ASSUMED]`, `[INFERRED]`, `[REQUIRES VALIDATION]`)

### Red Flags (Slop Indicators)

- ❌ "More" without specifics ("more scalable", "more reliable", "more performant")
- ❌ Buzzwords without definition ("cloud-native", "AI-powered", "best practices")
- ❌ Confident but unsubstantiated claims ("this approach is proven in production")
- ❌ Invented technologies or capabilities (technology or feature doesn't exist)
- ❌ Repeated content (same statement appears multiple times)
- ❌ Out-of-scope details (implementation tactics in architecture doc, business justification in technical spec)
- ❌ Unmarked inferences ("we'll need a DBA" without saying you assumed a dedicated team)
- ❌ Confidence inflation (medium-confidence estimate presented as high-confidence fact)

---

## Implementation Patterns

### Pattern 1: Quality Gate in Workflow

Insert anti-slop review **before** team review:
1. Agent generates documentation
2. Run anti-slop review
3. If <3 issues flagged: proceed to team review
4. If ≥3 issues flagged: send back to agent for revision or mark for human refinement

### Pattern 2: Iterative Refinement

1. Generate initial documentation
2. Run anti-slop review (e.g., "remove generic filler")
3. AI refines based on feedback
4. Run constraint grounding validation
5. Run hallucination detection
6. Proceed to team review when all gates pass

### Pattern 3: Confidence Scoring for Risk Management

1. Generate architecture + risk register
2. Run confidence scoring on all probability/impact estimates
3. For "low confidence" items: assign investigation task (collect more data, run spike, consult expert)
4. For "medium confidence" items: include in monitoring plan
5. For "high confidence" items: proceed with monitoring only

---

## Next Steps

After governance review:
1. **Archive the review checklists** — attach anti-slop review results to final architecture documentation
2. **Document your slop patterns** — create org-specific anti-slop guidance based on issues found
3. **Distribute guidelines** — share anti-slop checklist + constraint grounding template with your team
4. **Plan audits** — schedule confidence scoring on high-impact architecture decisions (quarterly or on major changes)

---

## Resources

- Anti-slop checklist: `/shared/anti-slop-checklist.md` (specific patterns to catch)
- Constraint-grounding rubric: `/shared/constraint-grounding-rubric.md` (how to verify constraints are respected)
- Hallucination detection guide: `/shared/hallucination-detection-guide.md` (patterns of invented tech, false confidence)
- Confidence scoring template: `/shared/confidence-scoring-template.md` (how to estimate and justify confidence levels)
- Document boundary reference: `/shared/document-scope-reference.md` (what belongs in ADRs, specs, requirements, etc.)

---

**Status:** Complete (6 prompt cards, validation checklists, customization knobs, anti-hallucination guardrails applied)
**Last Updated:** 2026-06-02 (Morpheus — Status Reconciliation)

