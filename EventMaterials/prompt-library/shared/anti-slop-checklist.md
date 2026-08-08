# Anti-Slop Checklist

A practical checklist to catch AI-generated hallucinations, generic filler, and ungrounded claims before sharing architecture documents.

---

## Purpose

AI is very good at producing confident-sounding but hollow text. This checklist helps you identify and fix the most common hallucination patterns.

---

## Hallucination Types to Catch

### ✓ Generic Adjectives Without Metrics

| Pattern | Weak | Strong |
|---------|------|--------|
| ❌ "More scalable" | ❌ "Our approach scales better" | ✓ "Scales from 1K to 100K concurrent users" |
| ❌ "Improved performance" | ❌ "Better latency" | ✓ "Latency reduced from 5s to 500ms p99" |
| ❌ "Easier to maintain" | ❌ "Better maintainability" | ✓ "Reduces MTTR by 50% (from 30min to 15min)" |
| ❌ "More reliable" | ❌ "Higher availability" | ✓ "Availability 99.5% → 99.99%" |
| ❌ "Simpler architecture" | ❌ "Reduced complexity" | ✓ "Eliminates 3 failure modes; reduces component count from 8 to 5" |

**Checklist:** For every adjective (better, faster, easier, simpler, more), ask: **What number would make this claim specific?** If no number exists, the claim is slop.

---

### ✓ Ungrounded Technology Claims

| Pattern | Weak | Strong |
|---------|------|--------|
| ❌ "Microservices scale better" | ❌ "Microservices are the right choice" | ✓ "Microservices enable 5 independent teams to deploy without coordination (req: fast feature velocity)" |
| ❌ "Cloud is more reliable" | ❌ "Use cloud services" | ✓ "AWS RDS Multi-AZ: automatic failover + 99.95% SLA = acceptable for 99.5% target" |
| ❌ "Kubernetes is industry best practice" | ❌ "Modern apps use Kubernetes" | ✓ "Kubernetes: enables independent deployments + auto-scaling; team learning 3 months + ops burden 1 FTE" |
| ❌ "NoSQL is faster" | ❌ "Use NoSQL for performance" | ✓ "DynamoDB: <100ms latency at scale; but eventual consistency violates order consistency requirement" |

**Checklist:** For every technology recommendation, ask: **How does this fit our specific constraints (team size, budget, timeline, SLA)?** If the answer references generic traits ("scalable," "modern"), it's slop.

---

### ✓ Invented Capabilities or Unspecified How-To

| Pattern | Weak | Strong |
|---------|------|--------|
| ❌ "AI-powered auto-scaling" | ❌ "Self-healing infrastructure" | ✓ "Kubernetes HPA: scales pods based on CPU >70%; tested with load test; scales from 3 to 20 pods" |
| ❌ "Seamless failover" | ❌ "Zero-downtime deployment" | ✓ "Blue-green deployments: 5-minute cutover; requires DNS update (tested)" |
| ❌ "Automatic error recovery" | ❌ "Self-correcting systems" | ✓ "Circuit breaker: 5 consecutive failures → fail-open; retries every 30s; manual override if needed" |

**Checklist:** For every claim of "automatic," ask: **Specifically, what detects the problem and what action does it take?** If you can't name the mechanism, it's invented.

---

### ✓ Repeated Content or Assumptions

| Pattern | Weak | Strong |
|---------|------|--------|
| ❌ Assumption mentioned 3 times in different sections | ✓ Mentioned once; referenced by ID in other sections |
| ❌ "Team must learn Kubernetes" (stated 2x) | ✓ "Team learning curve: Kubernetes (3 months)" (stated once in constraints; referenced as ASSUME-001) |
| ❌ "Availability 99.5%" repeated in multiple sentences | ✓ "Availability target: 99.5% (from Constraints)" (stated once; referenced elsewhere) |

**Checklist:** For every statement, ask: **Have I said this already?** Use find/replace to check for repeated phrases. Combine into one clear statement; remove duplicates.

---

### ✓ Out-of-Scope Details

| Document Type | Belongs | Doesn't Belong |
|-------------|---------|-----------------|
| **ADR** | Why decision made; tradeoffs | Implementation details (code patterns, library choices) |
| **Architecture Spec** | Components, responsibilities, data flows | SQL query optimization; specific config values |
| **Process Flow** | Steps, decision points, error handling | Business process details (when inventory is counted) |
| **Risk Register** | Risk, probability, mitigation, monitoring | Incident postmortem specifics (what log line said) |

**Checklist:** For each section, ask: **Is this detail critical for a reader to understand the architecture decision?** If it's implementation nitty-grit, move it to a separate doc or implementation wiki.

---

### ✓ Ungrounded Confidence

| Pattern | Weak | Strong |
|---------|------|--------|
| ❌ "This design guarantees 99.99% uptime" | ✓ "This design supports 99.5% uptime if: (1) database failover works, (2) on-call responds in <5 min, (3) monitoring catches issues within 30s. All tested. Confidence: 4/5." |
| ❌ "This eliminates all scaling concerns" | ✓ "Scales to 100K concurrent users; bottleneck appears at 1M concurrent (database write throughput); mitigated by sharding." |
| ❌ "Proven solution" | ✓ "Used by 3 similar projects (e-commerce, SaaS, mobile backend); published in [case study URL]" |

**Checklist:** For every claim of reliability/scaling/effectiveness, ask: **What could go wrong? What assumptions must hold for this to work?** If you can't list assumptions and failure modes, the claim is ungrounded.

---

## Pre-Review Checklist

Run this before sharing any architecture document with stakeholders:

### 1. **Generic Adjectives**
- [ ] Search document for: "better," "faster," "easier," "simpler," "more reliable"
- [ ] For each, replace with measurable claim (e.g., "50ms latency improvement") OR delete
- [ ] Result: Zero instances of unsupported adjectives

### 2. **Ungrounded Recommendations**
- [ ] Search for technology names (Kubernetes, microservices, NoSQL, serverless, etc.)
- [ ] For each, verify it references constraints (e.g., "team has 3 engineers → managed service preferred")
- [ ] Result: Every tech choice grounded in stated constraints

### 3. **Invented Capabilities**
- [ ] Search for: "automatic," "seamless," "zero-," "self-"
- [ ] For each, specify: What detects the problem? What action does it take? Has it been tested?
- [ ] Result: No invented capabilities; only named mechanisms

### 4. **Repeated Content**
- [ ] Use find/replace for key terms (team size, SLA, constraint names, assumption names)
- [ ] Identify duplicates; combine into single statement with references
- [ ] Result: Each fact stated once; referenced by ID elsewhere

### 5. **Out-of-Scope Details**
- [ ] ADRs: No code examples or library recommendations
- [ ] Specs: No SQL optimization details or config values
- [ ] Flows: No business logic; only operational sequence
- [ ] Result: Each document stays focused on its purpose

### 6. **Ungrounded Confidence**
- [ ] Search for: "guaranteed," "ensures," "proven," "industry standard"
- [ ] For each, replace with: Conditions (if X, Y, Z), Assumptions (marked), Confidence (1–5)
- [ ] Result: Confidence qualified by explicit conditions

### 7. **Missing Evidence**
- [ ] For every major recommendation, ask: What data supports this?
- [ ] If no data: Mark as [ASSUMPTION], add to assumption register, plan validation
- [ ] Result: No claims without supporting evidence or explicit assumption flag

### 8. **Missing Constraints Reference**
- [ ] For every decision, verify it references stated constraints (team size, budget, SLA, timeline)
- [ ] If missing: Add constraint reference or mark as speculative
- [ ] Result: Every recommendation tied to specific constraint

---

## Slop Scorecard

**Rate each document on slop patterns:**

| Pattern | Count | Severity | Action |
|---------|-------|----------|--------|
| Generic adjectives (no metrics) | 7 | High | Replace all with measurements |
| Ungrounded tech choices | 2 | High | Add constraint references |
| Invented capabilities | 1 | Critical | Remove or specify mechanism |
| Repeated assumptions | 3 | Medium | Consolidate; use references |
| Out-of-scope details | 5 | Medium | Move to implementation docs |
| Ungrounded confidence claims | 2 | High | Qualify with conditions + assumptions |
| Missing evidence | 4 | Medium | Add assumption flags; plan validation |

**Score:** 24 instances of slop detected. **Action:** Revise before sharing.

---

## Common Slop Phrases (Find & Fix)

| Slop Phrase | Problem | Fix |
|------------|---------|-----|
| "Scales well" | Generic | "Scales to 100K concurrent users" |
| "Industry best practice" | Generic + ungrounded | "Used by X% of similar projects; [reasoning specific to constraints]" |
| "Proven technology" | Generic + unverified | "Maturity level 5 (stable); used by [N] companies in similar domain" |
| "Handles growth" | Vague | "Handles 10x growth; bottleneck at 100x (mitigated by X)" |
| "Improves reliability" | Ungrounded | "Reduces failure rate from X to Y due to [specific mechanism]" |
| "Better than alternative" | Comparison without grounding | "Better fit because [specific constraint]; alternative violates [constraint]" |
| "Ensures quality" | Invented guarantee | "Quality checked via [specific process]; MTTR = [measured value]" |
| "Modern architecture" | Buzzword | Name specific characteristics: "Stateless, event-driven, independently deployable services" |
| "Cloud-native" | Buzzword | Specific technologies: "Containerized on Kubernetes, managed data stores, auto-scaling" |
| "Seamless integration" | Invented capability | "Integrates via REST API; timeout 5s; circuit breaker if >3 failures" |

---

## Peer Review Feedback Template

**When reviewing peer's architecture document, use this structure:**

```
SLOP FOUND:
1. [Line X] "Better performance" — Replace with specific latency improvement (current: ?, target: ?)
2. [Line Y] "Microservices scale better" — Add: Scale to WHAT? Why? How does this fit our 3-person team?
3. [Line Z] Repeated assumption 3x: "Team learns Kubernetes" — Consolidate to single mention + reference

UNGROUNDED CONFIDENCE:
1. "Guarantees 99.99% uptime" — Specify conditions: failover works, monitoring detects issues in 30s, on-call <5 min response
2. Missing assumption register — What could go wrong? Add [ASSUMPTION] flags + validation plan

MISSING EVIDENCE:
1. "Kubernetes is right choice" — What projects succeeded with Kubernetes? What team size? What timeline?
2. Cost claim unsupported — Where's the $X/month infrastructure cost breakdown?

RECOMMENDATIONS:
- Replace 5 generic adjectives with measurements
- Add constraint references for 3 tech choices
- Consolidate 3 repeated assumptions into 1 with references
- Result: Ready for CTO review
```

---

## Anti-Slop Culture

**Team practices to catch slop early:**

1. **Peer review before CTO review:** Catch obvious slop before it reaches leadership
2. **Template compliance:** Use provided templates; deviation requires justification
3. **Assumption registration:** Every unvalidated claim becomes assumption register entry
4. **Evidence requirement:** "Where's the evidence?" becomes standard question
5. **Constraint reference:** Every recommendation must cite a stated constraint
6. **Confidence scoring:** Claims qualified by confidence (1–5) + conditions
7. **Red team practice:** Tank plays skeptic; catches holes before formal review

---

## Quick Slop Detection (Red Line)

**If a sentence contains ANY of these, it's slop:**

- ❌ Adjective without number (better, faster, simpler, easier, more)
- ❌ Superlative (best, most, worst, greatest)
- ❌ "Automatic" without mechanism
- ❌ "Ensures" or "guarantees" without conditions
- ❌ Same fact repeated twice
- ❌ "Industry standard" without data
- ❌ Implementation detail in architecture document
- ❌ Confidence claim without assumption list

**Before final approval:** Zero red lines remain.
