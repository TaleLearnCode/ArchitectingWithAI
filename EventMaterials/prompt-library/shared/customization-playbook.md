# Prompt Customization Playbook

*How to adapt library prompts for your context, constraints, and team.*

---

## Overview

The prompt library provides canonical templates designed for generalist architecture scenarios. Real-world projects demand customization. This playbook walks you through systematically adapting any prompt to your **domain**, **scale**, **constraints**, and **audience**.

**Key Principle:** Customization is not optional tweaks—it's the difference between a prompt that works and a prompt that wastes your time.

---

## Section 1: Constraint-Driven Customization

Every architectural decision depends on constraints. Library prompts assume generic constraints; your project has specific ones.

### Step 1a: Identify Your Constraints

Before customizing any prompt, document your constraints explicitly:

| Constraint Category | Questions | Example Answers |
|-------------------|-----------|-----------------|
| **Team Size & Skill** | How many engineers? DevOps maturity? Language expertise? | 5 engineers; basic Kubernetes; strong Python |
| **Deployment Environment** | On-prem? Cloud? Hybrid? Edge? | AWS; single region; no local data residency |
| **Regulatory & Compliance** | GDPR? HIPAA? PCI-DSS? SOC2? | SOC2 required; no EU data processing |
| **Performance Targets** | Latency? Throughput? Consistency? | <500ms p99; 10K req/s; eventual consistency OK |
| **Budget & Operational Overhead** | Total infrastructure spend? On-call burden? | $50K/month max; 2 on-call engineers |
| **Time-to-Market** | Proof-of-concept? MVP? Production? | MVP in 3 months; iterate monthly |
| **Data Characteristics** | Volume? Velocity? Variety? Consistency needs? | 1TB/day; real-time stream; mixed data types; ACID required for subset |
| **Integration Scope** | How many external systems? Frequency? Failure tolerance? | 12 vendor APIs; synchronous; 99.5% uptime required |
| **Organizational Risk Appetite** | Greenfield vs. legacy? Innovation vs. stability? | Greenfield; medium innovation appetite; avoid unfamiliar tech |

### Step 1b: Layer Constraints into Prompts

**Pattern:** Replace generic statements with constraint references.

**Generic Prompt Statement:**
```text
Propose 4 distinct architecture approaches for this problem.
```

**Customized Prompt Statement:**
```text
Given these constraints:
- Team: 5 engineers, basic Kubernetes, strong Python
- Deployment: AWS, single region
- Performance: <500ms p99, eventual consistency OK
- Budget: $50K/month max
- Time-to-market: MVP in 3 months

Propose only architecture approaches that fit ALL constraints.
For each option, explain why it fits or explain why it doesn't.
Reject any approach that violates a constraint.
```

### Step 1c: Define What "Fits" Means

For each constraint, explicitly define acceptance criteria:

**Example: Team Size Constraint**
```text
CONSTRAINT: 5 engineers; DevOps maturity = basic

FITNESS DEFINITION:
- ✓ Does NOT require dedicated platform engineering team
- ✓ Can be operated by on-call rotation (2–3 engineers per shift)
- ✓ Failure modes are debuggable by generalist engineers in <1 hour
- ✗ Requires advanced Kubernetes operators or service mesh expertise
- ✗ Requires 24/7 dedicated infrastructure team
```

**Example: Budget Constraint**
```text
CONSTRAINT: $50K/month infrastructure max

FITNESS DEFINITION:
- ✓ Each architecture option includes estimated monthly cost
- ✓ Costs grounded in published AWS/GCP/Azure pricing (as of today)
- ✓ Contingency reserve calculated (20% buffer)
- ✗ Estimated costs are invented or aspirational
```

---

## Section 2: Domain-Specific Prompt Tailoring

Library prompts apply generic language. Your domain often has specific patterns, vocabularies, and risk profiles.

### Step 2a: Map Domain-Specific Risks

Identify which risks are domain-relevant:

| Domain | Primary Risks | Secondary Risks | Anti-Patterns |
|--------|---------------|-----------------|----------------|
| **E-Commerce** | Data consistency (inventory), payment reliability, fraud, cart loss | Seasonal load spikes, international compliance | Over-engineering single points of truth |
| **FinTech** | Auditability, regulatory compliance, data accuracy, fraud prevention | Cross-border settlement, latency under load | Misunderstanding eventual consistency for ledger data |
| **Healthcare** | HIPAA compliance, audit trails, patient privacy, uptime SLAs | Data residency (per state), integration with legacy systems | Assuming cloud-only is compliant without attestation |
| **SaaS/Multi-Tenant** | Noisy neighbor problem, data isolation, compliance per customer, billing accuracy | Scaling metadata layer, cross-tenant analytics | Building row-level security instead of data partitioning |
| **Real-Time Analytics** | Data freshness, query performance, cost of compute, data correctness | Backfill complexity, SLA vs. cost tradeoff | Assuming all data needs real-time freshness |
| **IoT/Edge** | Connectivity resilience, data sovereignty, edge computation, bandwidth | Time-series data volume, cold-storage cost | Assuming cloud-first for edge-native scenarios |
| **ML/AI Pipeline** | Reproducibility, feature freshness, model serving latency, data lineage | Training cost, experimentation velocity, feature store consistency | Underestimating data engineering complexity |
| **Content Distribution** | Cache invalidation, regional consistency, CDN bandwidth, media serving | DRM/licensing, transcoding compute, storage archival | Treating all content as if it has identical freshness/consistency needs |

### Step 2b: Inject Domain Knowledge into Prompts

**Pattern:** Add domain-specific context before the generic prompt.

**Generic Architecture Review:**
```text
Simulate an Architecture Review Board.
Roles: CTO, CISO, SRE Lead, Principal Architect, Finance Director.
Each role must ask hard questions and challenge assumptions.
```

**Domain-Customized (E-Commerce) Architecture Review:**
```text
Simulate an Architecture Review Board for an e-commerce platform.

Roles & Domain Focus:
- CTO: Asks about data consistency guarantees (inventory accuracy, double-charge prevention)
- CISO: Focuses on payment compliance (PCI-DSS), fraud detection, customer data isolation
- SRE Lead: Challenges scalability during seasonal peaks (Black Friday 10x traffic)
- Principal Architect: Validates integration points with payment processors, shipping APIs, analytics
- CFO: Questions infrastructure cost per transaction; tolerance for peak-season overspending

Domain-Specific Hard Questions You MUST Ask:
1. How does inventory consistency guarantee prevent overselling during traffic spikes?
2. What happens if the payment processor returns ambiguous success/failure?
3. How is cross-region inventory synchronized? Can it cause customer-visible stale data?
4. What is the cost per transaction? How does it scale?
5. How quickly can you recover if your database is corrupted? Backup strategy?

Domain-Specific Anti-Patterns to Challenge:
- Assuming "eventual consistency everywhere" is acceptable for inventory
- Over-reliance on a single payment processor without failover
- Storing customer PII and payment data in the same database
```

### Step 2c: Add Domain-Specific Validation Checkpoints

**Pattern:** Create a checklist of domain-specific acceptance criteria.

**Example: E-Commerce Architecture Validation**
```text
## Domain-Specific Validation Checklist

- [ ] Inventory system can handle concurrent updates from multiple order processors
- [ ] Payment processor failures do NOT block inventory updates
- [ ] Customer can see payment status within 5 seconds of charge
- [ ] PCI-DSS compliance verified by architecture (e.g., payment data never touches your logs)
- [ ] Seasonal 10x traffic spike does NOT cause inventory inconsistency
- [ ] Fraud detection can reject a transaction in <100ms
- [ ] Refund flow can reverse inventory updates atomically
- [ ] Analytics queries do NOT impact real-time transaction latency
- [ ] Backup/restore procedure tested; can restore inventory to any point in time
- [ ] Cross-region replication policy explicitly stated (and justified)
```

---

## Section 3: Audience-Aware Prompt Adaptation

Different audiences (executives, engineers, compliance teams) need different prompt framings.

### Step 3a: Identify Your Audience & Goals

| Audience | Primary Goal | Metrics They Care About | Tone | Level of Detail |
|----------|-------------|----------------------|------|-------------------|
| **Technical Team** | Build something that works, minimize rework | Technical correctness, time-to-implement, operational burden | Precise, matter-of-fact | High technical depth |
| **Executive Sponsor** | Understand business impact, cost, risk | ROI, time-to-market, competitive advantage, risk mitigation | Outcome-focused, concise | Summary + executive summary |
| **Compliance/Security** | Verify control coverage, risk mitigation | Control effectiveness, audit trail, exception handling | Evidence-based, conservative | Detailed with justifications |
| **Operations/SRE** | Keep it running, troubleshoot quickly | Observability, runbooks, failure modes, MTTR | Operational, pragmatic | Procedures + edge cases |

### Step 3b: Customize Prompts for Audience

**Pattern:** Add audience context and success criteria.

**Generic Prompt:**
```text
Generate an Architecture Specifications document.
Include: Component responsibilities, interfaces, C4 diagrams, NFR targets, technology choices.
```

**Audience-Customized (Executive):**
```text
Generate a 1-page Architecture Executive Summary for the executive sponsor.

MUST INCLUDE:
- Business problem being solved (in business terms, not technical)
- Architecture choice and why it was selected
- Top 3 business benefits
- Top 3 risks and mitigation strategy
- Estimated cost & time-to-implement
- Comparison to alternatives (1 sentence each)

DO NOT INCLUDE:
- Component diagrams or C4 models
- Technology brand names without business justification
- Implementation details
- Technical jargon without explanation

SUCCESS CRITERIA:
- Executive can explain to their boss why this architecture is chosen (in 2 minutes)
- Sponsor understands cost, timeline, and risk without asking technical questions
```

**Audience-Customized (Compliance Team):**
```text
Generate an Architecture Compliance & Control Matrix.

For each component, provide:
- Component name and responsibility
- Data it touches (classify by sensitivity: PII/PHI/Secrets/Public)
- Security controls (authentication, encryption, audit logging)
- Compliance requirements it addresses (e.g., HIPAA requires audit trail for all PHI access)
- Known exceptions (e.g., "Secrets stored in environment variables [ACCEPTED RISK: mitigation is limited deployment team access]")

DO NOT INCLUDE:
- Performance characteristics
- Cost estimates
- Deployment timelines
- Technology brand preferences without compliance rationale

SUCCESS CRITERIA:
- Compliance officer can determine: "Do I approve this for production?"
- Auditor can verify control coverage without asking architects questions
- You can trace every data element to the control protecting it
```

**Audience-Customized (Operations/SRE):**
```text
Generate a "Day 2 Operations" runbook for this architecture.

For each component:
- How to monitor it (metrics, thresholds, alerts)
- Common failure modes (and how to detect them)
- Troubleshooting procedure (step-by-step)
- Recovery procedures (both quick and comprehensive)
- Scaling procedure (when to add capacity)
- Backup/restore procedure (manual steps and verification)

CRITICAL PATHS:
- Identify dependencies; explain what breaks if each component fails
- Document state that must be preserved (databases, caches, configuration)
- Specify on-call escalation (when to page someone)

DO NOT INCLUDE:
- Architectural trade-offs
- Design rationale
- Implementation complexity
- Technology background

SUCCESS CRITERIA:
- An engineer new to this system can operate it without asking the architects
- Common failures can be resolved in <30 minutes using this runbook
- "It's broken" can be debugged in <5 minutes using diagnostics section
```

---

## Section 4: Iteration & Refinement

Library prompts rarely produce perfect output on the first run. Customize the feedback loop.

### Step 4a: Define Acceptance Criteria Before Running Prompt

**Pattern:** Create a checklist before prompting, not after.

**Example: Architecture Options Acceptance Criteria**
```text
BEFORE RUNNING PROMPT, I will accept the output only if:

STRUCTURAL:
- [ ] 3–5 distinct architecture options (not variations on one theme)
- [ ] Each option has documented trade-offs (not just pros)

CONSTRAINT-FIT:
- [ ] Each option explicitly explains how it fits the team-size constraint
- [ ] Each option includes estimated monthly infrastructure cost
- [ ] Each option explains operational burden (on-call, incident response)

DOMAIN-SPECIFIC:
- [ ] For e-commerce: each option addresses inventory consistency strategy
- [ ] For e-commerce: payment processor integration discussed
- [ ] For e-commerce: seasonal scaling strategy explained

CONFIDENCE:
- [ ] No confidence scores below 3 without explanation
- [ ] Any invented assumptions marked [INFERRED]
- [ ] Any inferred assumptions stated explicitly

TRACEABILITY:
- [ ] Every recommendation can be traced to a constraint or requirement
- [ ] Recommendations that change if constraints change are explicitly called out
```

### Step 4b: Review & Iterate with Guardrails

**Pattern:** Review output systematically; rerun if guardrails are violated.

**Systematic Review Checklist:**
```text
1. HALLUCINATION CHECK
   - Does output reference constraints I provided? (Yes/No)
   - Does output invent constraints not in my input? (Yes/No — if yes, mark as [INFERRED])
   - Can I trace every claim back to my requirements? (Yes/No)

2. CONFIDENCE CHECK
   - Are there statements with no basis? (Yes/No — if yes, mark as [SPECULATIVE])
   - Are confidence scores grounded in data? (Yes/No)
   - Are there buzzwords without definition? (Yes/No)

3. COMPLETENESS CHECK
   - Is the prompt response addressing my question? (Yes/No)
   - Are there gaps that matter for my decision? (Yes/No — if yes, note them)
   - Is the level of detail appropriate? (Too shallow/Too detailed/Just right)

4. USABILITY CHECK
   - Can I act on this without asking follow-up questions? (Yes/No)
   - Do I have all the information I need to make a decision? (Yes/No)
   - Is there irrelevant information I should discard? (Yes/No)
```

If review fails, rerun with refined prompt:

**If Hallucination Detected:**
```text
FEEDBACK: Your output included [INFERRED] assumptions I didn't provide:
- You assumed we use microservices, but I didn't specify this
- You assumed 24/7 on-call coverage, but I specified 2 shifts

REFINED PROMPT:
Do NOT invent assumptions. If information is missing, output [INPUT MISSING: <field>] and explain why it matters.
Proceed ONLY if all required inputs are provided.
```

**If Confidence Is Low:**
```text
FEEDBACK: Your confidence scores lack grounding:
- You rated "Cost: 3/5" but didn't state what you based this on
- The budget constraint I provided is clear ($50K/month), so cost confidence should be higher

REFINED PROMPT:
For each rating, explain the basis:
Example: "Cost: 4/5. GROUNDED IN: Published AWS pricing for RDS + EC2 + S3 (as of 2026-06)."
If you're guessing, output [UNKNOWN] instead of a number.
```

**If Output Is Incomplete:**
```text
FEEDBACK: Missing information:
- You didn't explain what happens if a domain registration fails
- You didn't address the seasonal traffic spike scenario

REFINED PROMPT:
Address these specific gaps:
1. Failure scenario: Domain registration service unavailable → What does the system do?
2. Seasonal scenario: Traffic 10x peak on Black Friday → Which components scale? Cost impact?
```

---

## Section 5: Quick Customization Template

Use this template to customize any library prompt in 5 minutes:

```markdown
## Customization Worksheet

**Prompt Title:** [Which library prompt are you customizing?]

### 1. Constraints (Required)
- Team size & skill level:
- Deployment environment:
- Budget & operational overhead:
- Performance targets:
- Regulatory requirements:

### 2. Domain Context (Optional but Recommended)
- What industry/domain? (E-commerce, SaaS, FinTech, etc.)
- What domain-specific risks matter most?
- What domain anti-patterns should we avoid?

### 3. Audience & Framing (Required)
- Who is this output for? (Technical team / Executive / Compliance / Ops)
- What question are they trying to answer?
- What metrics do they care about?

### 4. Acceptance Criteria (Required)
- Structural: What counts as a complete response?
- Constraint-fit: How do I know it fits my constraints?
- Domain-specific: What domain-specific checks apply?
- Confidence: What confidence threshold do I accept?

### 5. Customization Edits (Track Here)
- Added to prompt: [list changes you made to the library prompt]
- Removed from prompt: [list what you removed as not relevant]
- Special instructions: [any custom guidance specific to your project]

### 6. After Running (Review Notes)
- What worked well?
- What was missing?
- What should I adjust for next time?
```

---

## Section 6: Common Customization Pitfalls

### Pitfall 1: Over-Constraining Too Early

**Mistake:** Adding so many constraints that AI cannot generate alternatives.

**Fix:** Distinguish between hard constraints (must have) and soft constraints (prefer).

```text
HARD CONSTRAINTS (No options violate these):
- Budget: Cannot exceed $50K/month
- Team size: Cannot require more than 5 engineers
- Compliance: Must be HIPAA-compliant

SOFT CONSTRAINTS (Options should prefer these):
- Prefer AWS over on-prem (but on-prem is acceptable if justified)
- Prefer open-source technologies (but proprietary is OK if ROI is clear)
- Prefer serverless where possible (but containers are acceptable)

OPTIONAL PREFERENCES:
- Python preferred language (but Java/Go acceptable)
- PostgreSQL preference (but schema-flexible databases OK for specific use cases)
```

### Pitfall 2: Vague Domain Context

**Mistake:** Saying "E-commerce platform" without specifying what kind.

**Fix:** Be specific about the domain variant.

```text
INSTEAD OF: "We're building an e-commerce platform"
BE SPECIFIC: "We're building a B2B marketplace for industrial equipment, 
            with 200 suppliers and 2,000 buyers. 
            Inventory is managed by suppliers (not centralized). 
            Orders process asynchronously. 
            Suppliers must be able to update inventory in real-time."
```

### Pitfall 3: Accepting Output Without Validation

**Mistake:** Running prompt once and treating output as truth.

**Fix:** Always validate output against acceptance criteria before acting.

```text
NEVER: "AI said microservices, so we're building microservices"
ALWAYS: "AI recommended microservices. Let me validate:
        - Does this fit our team size? [Check constraints]
        - Is this based on our requirements or invented? [Check hallucinations]
        - What's the confidence level? [Check grounding]
        - What happens if we're wrong? [Check risk mitigation]"
```

### Pitfall 4: Assuming One-Size-Fits-All Prompt

**Mistake:** Using the same prompt for multiple team members, all expecting same output.

**Fix:** Customize prompts per audience; acknowledge output will differ.

```text
SCENARIO: You're presenting architecture options to:
- Technical team (needs deep implementation details)
- CFO (needs cost analysis)
- CISO (needs security controls)

SOLUTION: Run THREE CUSTOMIZED PROMPTS:
1. "Generate architecture options with implementation complexity & team ramp-up time"
2. "Generate architecture options with cost-per-transaction & infrastructure spend"
3. "Generate architecture options with security controls & compliance implications"

Different outputs? That's expected. They're answering different questions.
```

---

## Checklist: Before Using Any Library Prompt

- [ ] I have documented my constraints explicitly
- [ ] I have mapped domain-specific risks and anti-patterns
- [ ] I have defined who the audience is and what success looks like
- [ ] I have written acceptance criteria (before running prompt)
- [ ] I have customized the prompt with my specific context
- [ ] I have a plan to validate output before acting on it
- [ ] I understand how my output would change if constraints changed

**When all boxes are checked, run the prompt with confidence.**

---

## Next Steps

1. **Pick one prompt** from the library that you want to use
2. **Work through this playbook** with your team
3. **Document your customization** using the worksheet in Section 5
4. **Validate the output** using acceptance criteria before acting
5. **Iterate** if output doesn't meet acceptance criteria
6. **Reflect:** What worked? What would you do differently next time?

The more you customize upfront, the better your output. Investment in clarity pays off in accuracy.
