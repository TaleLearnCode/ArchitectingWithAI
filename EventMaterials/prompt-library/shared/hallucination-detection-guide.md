# Hallucination Detection Guide

Checklist and patterns for identifying when AI has generated plausible-sounding but false or baseless content.

---

## Purpose

Hallucinations are confident-sounding but ungrounded claims. This guide teaches you to recognize them before they spread into decision documents.

---

## Hallucination Detection Patterns

### Pattern 1: Confident Claim with Zero Evidence

**Example:**
```
"React enables real-time collaboration features through its virtual DOM and hooks API."
```

**Why it's a hallucination:**
- Makes a causal claim (React enables collaboration)
- React enables collaboration no more than Node.js enables payment processing
- Conflates unrelated concepts
- Sounds authoritative but is backwards

**Detection:**
- Claim sounds technical and confident
- But making a causal leap (A enables B when A and B are orthogonal)
- No evidence provided

**Fix:**
```
"React is a UI library that efficiently updates views based on state changes (virtual DOM + reconciliation). Real-time collaboration features require a separate collaboration layer (e.g., Operational Transform, CRDT) regardless of UI framework. Technology choice (React vs. Vue vs. Angular) doesn't affect collaboration architecture."
```

---

### Pattern 2: Superlative Without Grounding

**Examples:**
```
"Microservices are the best architecture for any large-scale system."
"Kubernetes is the gold standard for container orchestration."
"GraphQL is superior to REST for all APIs."
```

**Why they're hallucinations:**
- "Best" and "gold standard" and "superior" are marketing language, not technical analysis
- Context-dependent: Best FOR WHAT? What constraints?
- No evidence for context-independent superiority
- Presented as objective truth when they're opinions

**Detection:**
- Superlatives (best, worst, greatest, superior, ideal, perfect)
- No mention of constraints or tradeoffs
- No acknowledgment of use cases where alternative is better

**Fix:**
```
"Microservices enable independent deployments (benefits: fast feature velocity, team autonomy). Tradeoff: Operational complexity + distributed system challenges. Best fit when: (1) Multiple independent teams, (2) Different deployment schedules, (3) Scale varies by service. Not best fit when: (1) Single team, (2) Coordinated releases acceptable, (3) Monolith scales fine."
```

---

### Pattern 3: Capability Claimed Without Mechanism

**Examples:**
```
"The system automatically scales to handle 10x traffic spikes."
"AI-powered recommendations improve user engagement by 40%."
"Self-healing infrastructure recovers from failures without intervention."
```

**Why they're hallucinations:**
- Mechanism is not explained
- Sounds like a feature but might not exist
- "Automatically" hides the real process
- The claimed effect (40% engagement) is not tied to the technology

**Detection:**
- Search for: "automatic," "self-," "AI-powered," "intelligent"
- Ask: "Specifically, how does this work?"
- If answer is vague, it's a hallucination

**Fix:**
```
"Scaling strategy: Kubernetes HPA (Horizontal Pod Autoscaler) monitors CPU usage. When CPU > 70% for 1 minute, adds pods; when CPU < 30%, removes pods. Tested: Simulated traffic spike; 10 new pods provisioned in 45 seconds. Not truly "automatic" in all failure scenarios (HPA can't help if database is bottleneck)."

"Recommendations: Content-based filtering (calculate similarity between products). Algorithm is deterministic, not AI/ML. Engagement improvement: Not measured yet; hypothesis is +10–20% CTR based on similar retailers (validate with A/B test)."

"Failure recovery: Pod crashes → Kubernetes restarts container. Application restarts → Graceful shutdown (drain connections, flush caches). Network partition → Circuit breaker fails open (manual intervention if >5 failures). 'Self-healing' only for specific failure types; manual ops needed for others."
```

---

### Pattern 4: Statistics Presented Without Source

**Examples:**
```
"90% of enterprises use microservices."
"Downtime costs $5,600 per minute."
"Cloud saves 30% on infrastructure costs."
```

**Why they're hallucinations:**
- Statistics sound precise but source unknown
- Often conflated from different contexts
- Presented as fact when they're estimates or studies (which might be outdated/biased)
- No link to your situation

**Detection:**
- Search for: percentage signs, dollar amounts, time figures
- Ask: "Where did this number come from?"
- If not attributed, it's hallucinated

**Fix:**
```
"Enterprise microservices adoption: Gartner reports ~60% of enterprises are adopting microservices as of 2023 (not 90%; hallucination likely came from vendor marketing). Caveat: Adoption ≠ success; many struggles with complexity."

"Downtime costs: IDC estimates $5,600 per minute for financial services (not universal). For e-commerce: ~$500–2,000 per minute depending on peak traffic. YOUR cost depends on revenue/minute during outage; calculate as (revenue/minute) × outage_duration_minutes."

"Cloud cost savings: Varies by use case. For compute-heavy work: 20–40% savings common. For bursty/variable workloads: 40–60% savings possible. For always-on baseline: on-premises might be cheaper. Calculate for your workload."
```

---

### Pattern 5: Technology Recommendation Without Tradeoff

**Examples:**
```
"Use Kubernetes for production deployments. It scales horizontally, provides load balancing, and enables zero-downtime deployments."
```

**Why it's a hallucination:**
- Lists only benefits
- No mention of costs (financial, operational, learning)
- No mention of simpler alternatives
- No mention of when NOT to use Kubernetes

**Detection:**
- One-way argument (only pros, no cons)
- No mention of constraints or tradeoffs
- Sounds like marketing copy

**Fix:**
```
"Kubernetes recommendation: Fits constraints IF (1) 5+ services need independent scaling, (2) Team has SRE expertise or budget for consulting, (3) $500K+ annual infrastructure budget. Tradeoffs: Learning curve 3–6 months, operational complexity high, cost scales with number of services. Alternative: Managed services (App Runner, AWS Lambda) for simpler use cases. Fit: Choose Kubernetes if independent service scaling is critical; choose managed services if simplicity matters more than control."
```

---

### Pattern 6: Invented Research or Case Studies

**Examples:**
```
"A study by leading researchers found that event-driven architectures improve latency by 50%."
"Fortune 500 companies report that DDD (Domain-Driven Design) prevents architectural mistakes."
"Research shows that TypeScript reduces bugs by 40%."
```

**Why they're hallucinations:**
- Study is vague ("leading researchers" but no name or link)
- Causal claim without context (latency improved in specific case, not universally)
- "Reports" without specific evidence
- Invented by AI to sound authoritative

**Detection:**
- Search for: "research shows," "studies find," "experts agree," "companies report"
- Ask: "Can you link to this study?"
- If no specific citation, it's hallucinated

**Fix:**
```
"Event-driven systems: In practice, event-driven improves throughput (decouples services) but adds latency (asynchronous, eventual consistency). Latency/throughput tradeoff depends on workload. Validate with load test before claiming improvement."

"DDD (Domain-Driven Design): Helps clarify architecture for complex domains (finance, healthcare). No magical prevention of mistakes; mistakes are possible with any architecture. Value: Better alignment between code and business logic. Not validated by Fortune 500 studies I can cite; opinion based on industry experience."

"TypeScript: Catches certain types of bugs at compile time (type errors). Doesn't prevent logic errors, architectural mistakes, or concurrency bugs. Bug reduction depends on team diligence + testing practices. 40% claim is marketing-speak; actual reduction varies (10–30% in experience reports)."
```

---

### Pattern 7: Vague Future Tense Claims

**Examples:**
```
"The system will automatically handle failures."
"The architecture will scale to support millions of users."
"The team will learn Kubernetes quickly."
```

**Why they're hallucinations:**
- "Will" is a prediction, not evidence
- No mention of how / by when / with what probability
- Presented as certainty when they're assumptions
- No fallback if prediction fails

**Detection:**
- Search for: "will," "can," "should," "likely"
- Ask: "What assumption is this based on? What if it's wrong?"
- If no validation plan, it's likely hallucinated

**Fix:**
```
"Failure handling: Currently, [list specific mechanisms in place: circuit breaker, timeouts, retry logic]. NOT automatic for all failure types. Gaps: [list what's not automated]. Automation improvements planned for Q2; timeline TBD."

"Scaling: Tested to 100K concurrent users (load test); projected to 1M based on [specific math: threadpool * instances * throughput]. Confidence: 3/5 (untested at 1M; assumes linear scaling holds). Validation: Monitor growth metrics; reassess at 500K concurrent."

"Team Kubernetes learning: Estimate 3 months based on similar team timelines. Confidence: 2/5 (high risk). Validation plan: Weekly checkpoint; if behind schedule by 2+ weeks, trigger fallback (App Runner instead of self-managed K8s)."
```

---

## Hallucination Checklist

**Before approving any document, scan for hallucinations:**

- [ ] No superlatives without grounding ("best," "superior," etc. all reference specific constraints)
- [ ] No invented mechanisms ("automatic" always followed by: what detects, what action, evidence)
- [ ] No statistics without sources (every number has [citation] or [ASSUMPTION] flag)
- [ ] No one-way arguments (every recommendation acknowledges tradeoffs + alternatives)
- [ ] No invented research (studies are named with URLs; quotes are attributed)
- [ ] No future tense certainty (predictions include confidence level + fallback)
- [ ] No vague claims ("handles," "supports," "enables" all specify how)

---

## Red Flags (High Probability of Hallucination)

| Red Flag | Hallucination Likely? | Action |
|----------|---------------------|--------|
| No citations or [ASSUMPTION] flags | Very likely | Demand evidence or mark as speculative |
| Superlatives (best, worst, superior) | Very likely | Reframe with constraints and tradeoffs |
| Marketing language ("modern," "scalable," "industry standard") | Very likely | Replace with specific capabilities |
| Mechanism claimed ("automatic," "self-healing") | Very likely | Ask: specifically, how? What detects + what acts? |
| Precise statistics without source | Likely | Demand source or label as estimate with confidence |
| One-way argument (only benefits) | Likely | Add tradeoffs + alternatives + when not to use |
| Invented research ("studies show," "research finds") | Likely | Demand specific citation (URL, author, date) |
| Future tense certainty ("will scale," "can handle") | Likely | Add confidence level + validation plan + fallback |

---

## Confidence-Based Hallucination Scoring

**For each suspicious claim, rate hallucination risk:**

| Risk Level | Indicators | Action |
|-----------|-----------|--------|
| **High (80%+ hallucination)** | Superlative + no evidence + marketing language | Reject or completely rewrite |
| **Medium-High (50–80%)** | Mechanism claimed but not specified; statistics without source | Demand specifics; mark [REQUIRES_VERIFICATION] |
| **Medium (30–50%)** | Prediction without confidence + fallback; vague mechanism | Add confidence level; clarify mechanism; add fallback |
| **Low (<30%)** | Specific mechanism named; evidence cited; confidence qualified | Review for completeness but low risk |

---

## Hallucination Recovery Procedure

**If you discover a hallucination in approved content:**

1. **Quarantine:** Mark the claim [HALLUCINATION_DETECTED]; note date discovered
2. **Understand:** What was hallucinated? Why did it seem plausible?
3. **Correct:** Rewrite using evidence-based reasoning (or mark [NEEDS_RESEARCH])
4. **Trace:** Where did hallucination come from? (AI? Human? Misunderstanding?)
5. **Communicate:** Alert stakeholders who saw the original (prevent downstream decisions based on false claim)
6. **Prevent:** Update approval process to catch this pattern earlier

---

## Hallucination Pattern Learning

**Track hallucinations discovered in your project:**

| Hallucination | Context | Pattern | Prevention |
|--------------|---------|---------|-----------|
| "Will scale to 1M users" (unsupported) | Architecture spec | Unqualified prediction | Add confidence + fallback to all predictions |
| "Reduces bugs by 40%" (sourced to nowhere) | Technology choice ADR | Invented statistic | Demand citation for all statistics |
| "Auto-scaling handles failures" (vague) | Process flow | Invented mechanism | Ask "specifically, how?" for all "auto" claims |

After 5–10 hallucinations, patterns emerge. Use patterns to improve your review checklist.

---

## Training Material: Spot the Hallucination

**Exercise: Which of these are hallucinations? Why?**

1. "Microservices are more scalable than monoliths."
2. "PostgreSQL can handle 100K concurrent connections; we tested it with our schema."
3. "The system automatically recovers from database failures."
4. "GraphQL is the best choice for mobile apps."
5. "RDS Multi-AZ provides automatic failover within 30 seconds (tested in staging); SLA 99.95%."
6. "Our team will learn Kubernetes in 3 months (confidence 2/5; high risk; fallback planned)."
7. "AI-powered recommendations improve engagement."
8. "API latency improved by 50% after adding caching (measured: 500ms → 250ms p99)."

**Answers:**
1. ❌ Hallucination (superlative without grounding; depends on use case)
2. ✓ Evidence-based (specific test described)
3. ❌ Hallucination (mechanism not described; what "automatically"? How detected?)
4. ❌ Hallucination (superlative; depends on constraints; one-way argument)
5. ✓ Evidence-based (specific mechanism + SLA + tested)
6. ✓ Evidence-based (prediction qualified with confidence + fallback)
7. ❌ Hallucination (mechanism not described; causation not established)
8. ✓ Evidence-based (specific measurement; before/after)

---

## Hallucination Prevention Culture

**Team practices to prevent hallucinations:**

1. **Cite everything:** Every recommendation must link to: (a) evidence (link/study), (b) constraint (named), or (c) [ASSUMPTION] flag
2. **Demand specificity:** "How specifically?" is the default question for every claim
3. **Red team review:** Tank (analyst) reviews every document for hallucinations before approval
4. **Assumption register:** Every unproven claim becomes an assumption with validation plan
5. **Evidence threshold:** "Prove it or flag it as speculative" is team norm
6. **Confidence scoring:** Predictions always include confidence level (1–5) + fallback
