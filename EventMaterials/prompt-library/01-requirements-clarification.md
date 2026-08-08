# 01 — Requirements Clarification Prompts

*Help architects uncover ambiguities, contradictions, hidden NFRs, and missing constraints before design begins.*

---

## Section Purpose

Requirements are where architectural decisions begin. Unclear requirements lead to misaligned architectures.

These prompts help you:
- **Identify ambiguities** in functional and non-functional requirements
- **Surface hidden constraints** (team size, deployment environment, regulatory, latency, budget, operational maturity)
- **Extract implicit NFRs** that stakeholders assume but haven't stated
- **Find contradictions** between stated goals and constraints
- **Ask clarifying questions** that save design time later

**Thesis Connection:** AI broadens exploration. Here, it explores the requirements space, identifying gaps before architects commit to a direction.

---

## Prompt Catalog

---

## ✓ Requirements Ambiguity Scan

### Use This When
Your team has written functional requirements or user stories but you suspect they're incomplete, contradictory, or use vague language. Use this to surface specific ambiguities and contradictions before architecture work begins.

### When NOT to Use
Don't use if requirements are already clear and fully specified.

### Prerequisites
- User stories or functional requirements (minimum 3; representative of the problem domain)
- Acceptance criteria or success definitions
- Known constraints (team size, budget, deployment environment, SLA targets if available)

**Validation Rule:** If user stories are missing, output will say `[INPUT MISSING: User Stories]`. Gather them first; do not proceed with assumptions.

---

### Input Example

```
REQUIREMENTS:

Story 1: As a customer, I want to upload my product catalog so that it's available for customers to browse.
- Acceptance: CSV/XML format supported
- Acceptance: Catalog "live" within 2 hours of upload
- Acceptance: Invalid rows flagged with errors

Story 2: As a product manager, I want search results in under 1 second so customers can quickly find items.
- Acceptance: Filters include category, price, brand
- Acceptance: Minimum 10 facets supported

Story 3: As a merchant, I want to sync inventory with my ERP system so I don't manually update stock.
- Acceptance: Real-time sync preferred
- Acceptance: Supports JSON webhooks from ERP

CONSTRAINTS:
- Team: 3 backend, 2 frontend, 1 QA (no specialized DBAs)
- Budget: $150K/year
- SLA: 99.5% availability (business-critical)
```

---

### The Prompt

```text
Analyze these requirements and identify specific ambiguities, contradictions, and missing information.

For each finding, explain:
1. The exact phrase or requirement that is unclear
2. What it could mean (list 2-3 interpretations)
3. Why each interpretation matters for architecture
4. What you'd need to know to resolve it

Categories to examine:
- Functional ambiguities (what is "live"? what does "real-time" mean? what counts as valid?)
- Non-functional implications (search < 1 second implies what about data freshness? caching strategy?)
- Operational gaps (who monitors the webhook? who handles failures? what's the rollback plan?)
- Data lifecycle gaps (how long is data retained? what about customer privacy? audit requirements?)
- Integration gaps (what happens if ERP is down? what format for webhook payloads?)
- Edge cases (what if upload has 100K rows? what if search gets no results? what if sync fails halfway?)

Format your response as a numbered list. For each item, use this structure:
[AMBIGUITY TYPE]: [Exact phrase or requirement]
Possible interpretations: [2-3 options]
Matters because: [Why this affects architecture]
Clarifying questions: [What to ask stakeholders]

Do NOT invent new requirements or suggest solutions. Only identify what is unclear.
Do NOT assume standard industry practices. Only work with what is stated.

***
[PASTE YOUR REQUIREMENTS HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Depth** | You want exhaustive analysis | Add: "Also examine assumptions about user behavior, team skills, and vendor capabilities" |
| **Focus** | You care about specific risk areas | Add: "Prioritize ambiguities related to data consistency and compliance" |
| **Team angle** | You want perspective from a role | Add: "Answer from the perspective of someone who will operate this system (SRE/DevOps)" |
| **Format** | You want structured output | Add: "Use a table with columns: Ambiguity, Interpretation 1, Interpretation 2, Architectural Impact" |
| **Scope** | You want to exclude certain areas | Add: "Skip UI/UX requirements; focus only on backend and data concerns" |

---

### Example Output (Strong)

```
[AMBIGUITY 1]: "Catalog live within 2 hours of upload"
Possible interpretations:
  - All customers see the updated catalog within 2 hours (strong consistency)
  - The batch process completes in 2 hours, but visibility to customers happens later (eventual consistency)
  - Incremental updates are visible as they process (rolling visibility)

Matters because: This determines database consistency model
  - Strong consistency → single database, no denormalization
  - Eventual consistency → event-driven, caching layers, search indices can lag
  - Rolling visibility → complex coordination between upload and search systems

Clarifying questions:
  - "When a merchant uploads 5 new products, when must a customer be able to find them in search?"
  - "Is 2 hours a hard SLA or a 'typically' SLA?"
  - "What's more important: fast upload feedback or customer visibility? (They might conflict)"

[AMBIGUITY 2]: "Search results in under 1 second"
Possible interpretations:
  - Response time for search API is <1 second (from request to response)
  - Time from user clicking search to results appearing on screen is <1 second
  - Includes network latency or excludes it?

Matters because: Affects caching strategy and data architecture
  - API-only: May need in-memory index (Redis, Elasticsearch)
  - Screen-to-result: Need to account for frontend overhead; relaxes backend requirement
  - Network-inclusive: Need edge caching or CDN

Clarifying question: "Measure 1-second from API call or from user action? And does this include network?"
```

---

### Validation Checklist

- [ ] **Specific citations:** Every ambiguity quotes the exact phrase that's unclear (not "vague language in general")
- [ ] **Multiple interpretations:** At least 2 plausible readings per ambiguity (shows it's genuinely unclear, not just pedantic)
- [ ] **Architecture consequence:** Each ambiguity shows *why* it matters for design (not just "this is unclear")
- [ ] **Grounded questions:** Clarifying questions are specific to THIS project (not generic "ask stakeholders")
- [ ] **No invented requirements:** All identified ambiguities come from the input (AI hasn't inferred new features)
- [ ] **Edge cases examined:** Output includes operational gaps, not just functional clarity

**Scoring:**
- **All checked:** Output is trustworthy. Use clarifying questions with your team.
- **1–2 unchecked:** Acceptable; iterate if needed.
- **3+ unchecked:** Rerun with clarification or focus: "Skip edge cases; focus on functional ambiguities only."

---

## ✓ Requirements Deep Dive

### Use This When
You have relatively clear functional requirements but need to extract *all* implicit non-functional requirements, domain rules, and operational constraints. Use this after ambiguity scan to ensure nothing is missed.

### When NOT to Use
Skip this if your team can't act on NFRs yet (requirements still in flux).

### Prerequisites
- Clarified requirements (use Ambiguity Scan first if requirements are messy)
- Business context: What is the product? Who are users? What problem are we solving?
- Known constraints: Team experience, deployment model, regulatory domain

**Validation Rule:** If requirements lack business context AND known constraints, output will say `[INPUT MISSING: Business Context]` and `[INPUT MISSING: Constraints]`. Gather those first. If context is thin, output will be full of [INFERRED] markers — that's OK; it signals what still needs clarification.

---

### Input Example

```
CLARIFIED REQUIREMENTS:

Functional:
- Merchants upload product catalogs (CSV/XML/JSON)
- Customers search products by name, category, price range, brand
- Results return in < 1 second with pagination (50 items/page)
- Inventory syncs with merchant's ERP via webhook (updates within 15 minutes of ERP change)
- Customers can wishlist products

Non-Functional (stated):
- 99.5% availability SLA
- No GDPR data transfers outside EU
- Compliance: No payment card data stored

Constraints:
- Team: 3 backend engineers, 2 frontend, 1 QA
- Tech stack: Node.js, PostgreSQL (team expertise)
- Deployment: AWS, self-managed
```

---

### The Prompt

```text
Extract ALL non-functional requirements, domain rules, and operational constraints from these requirements.

Work systematically across these categories:

PERFORMANCE & SCALABILITY
- Latency targets (user-facing and background)
- Throughput targets (transactions/sec, concurrent users, data volume)
- Peak load assumptions (time of day, seasonal, growth projections)

RELIABILITY & AVAILABILITY
- Uptime targets (SLA %, how measured?)
- Mean time to recover (MTTR) targets
- Acceptable data loss or inconsistency (RPO/RTO)
- Failure scenarios (what if ERP is down? search index is stale?)

SECURITY & COMPLIANCE
- Data classification (public, sensitive, regulated?)
- Regulatory domain (GDPR, HIPAA, SOC2, PCI?)
- Access control requirements (who can see what data?)
- Audit/logging requirements

DATA LIFECYCLE
- Retention policies (how long kept?)
- Archival strategy (when moved to cold storage?)
- Deletion/anonymization requirements

OPERATIONAL CONSTRAINTS
- Deployment model (cloud, on-prem, hybrid?)
- Infrastructure limits (multi-region? single region?)
- Team skills and maturity (can they operate Kubernetes? Do they prefer managed services?)
- Maintenance windows (zero-downtime required? acceptable?)
- Monitoring & alerting (who watches? what's critical? what's nice-to-have?)

INTEGRATION CONSTRAINTS
- Dependency reliability (what if vendor API is down?)
- Frequency of sync (real-time vs. batch?)
- Data format standards (proprietary vs. industry standard?)

For each category, list:
- Explicit requirements (stated in input)
- [INFERRED] requirements (strongly implied by functional requirements or business model)
- [MISSING] requirements (critical gaps that need clarification)

Example [INFERRED]: "Search must be idempotent" inferred from fact that catalog can update while searches are in-flight
Example [MISSING]: "Catalog size" — affects choice of search technology (SQL? Elasticsearch? Algolia?)

***
[PASTE YOUR CLARIFIED REQUIREMENTS HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Rigor** | You want exhaustive coverage | Add: "For each [INFERRED] item, explain the reasoning and what would change if this assumption is wrong" |
| **Risk focus** | You're in a regulated industry | Add: "Prioritize security, compliance, and audit requirements; others are secondary" |
| **Team perspective** | You want ops-focused view | Add: "Emphasize operational constraints, maintenance burden, and monitoring/alerting needs" |
| **Business filter** | You want only business-critical NFRs | Add: "Flag each requirement as CRITICAL, IMPORTANT, or NICE-TO-HAVE based on business impact" |

---

### Example Output (Strong)

```
EXPLICIT REQUIREMENTS:

Performance:
✓ Search < 1 second (p95, implied by user experience requirement)
✓ Inventory sync within 15 minutes (stated in webhook requirement)
✓ 99.5% availability SLA (explicit)

Data Lifecycle:
✓ No GDPR transfers outside EU (explicit)
✓ No payment card storage (explicit)

INFERRED REQUIREMENTS:

Reliability:
[INFERRED] Search index can be stale (since catalog updates may take 2 hours, search won't always show latest)
  - Consequence: Must communicate eventual consistency to users ("some inventory may be cached")
  - Must validate: Is staleness acceptable? If not, need immediate-update path

[INFERRED] Webhook failure recovery needed (if ERP webhook fails, inventory won't sync)
  - Consequence: Need retry logic, dead-letter queue, or fallback to polling
  - Must validate: What happens if sync fails? Manual recovery? Auto-recovery?

Operational:
[INFERRED] On-demand scaling required (merchants might upload 100K-item catalogs; search spikes possible)
  - Consequence: Database must support scaling (read replicas, caching, or cloud DB with auto-scaling)
  - Implies: Cannot use single-machine PostgreSQL in production

[INFERRED] 24/7 monitoring needed (99.5% SLA implies proactive alerting)
  - Consequence: Observability required (metrics, logs, distributed tracing)
  - Team impact: Ops on-call? Or managed service?

MISSING REQUIREMENTS (Clarify with Stakeholders):

Critical:
❓ Peak concurrent users? (Affects database connection pooling, cache sizing)
❓ Catalog size? (100 products? 1M products? Affects search architecture)
❓ Search result freshness tolerance? (1 minute old? 1 hour old? Real-time only?)

Important:
❓ Disaster recovery RTO/RPO? (If database fails, how long acceptable downtime? Data loss?)
❓ Geographic redundancy needed? (Single-region or multi-region?)
❓ Customer data retention? (Delete old wishlists? Archive inactive customers?)

Nice-to-Have:
❓ Analytics on search behavior? (Affects logging and warehousing)
❓ A/B testing on search algorithm? (Affects data pipeline)
```

---

### Validation Checklist

- [ ] **Categories covered:** Output addresses performance, reliability, compliance, operational, and integration dimensions (not just functional)
- [ ] **Inference marked:** Every [INFERRED] item clearly shows what was assumed and why
- [ ] **No invented features:** [INFERRED] requirements are plausible consequences of stated requirements, not new ideas
- [ ] **Missing items concrete:** [MISSING] items are specific questions, not vague ("more clarity needed")
- [ ] **Traceability:** Every explicit requirement can be traced to input; inferences state their basis

---

## ✓ Stakeholder Perspective Simulation

### Use This When
You want to uncover hidden requirements and concerns from non-technical stakeholders or roles not present in requirements discussions. This surfaces disagreement and blind spots early.

### When NOT to Use
Don't use if you have direct access to all stakeholders (run live meetings instead).

### Prerequisites
- Current requirements draft (even if messy)
- List of key stakeholder roles (Product Owner, Security, SRE, CFO, Compliance, Legal, End User, etc.)
- Familiarity with their KPIs or goals (what success looks like for each role)

**Validation Rule:** If requirements are missing AND stakeholder roles are missing, output will say `[INPUT MISSING: Stakeholder Roles]` and `[INPUT MISSING: Requirements]`. Gather both first. If only stakeholder goals are unknown, you'll get generic concerns — that's OK; use output to drive conversations with actual stakeholders.

---

### Input Example

```
REQUIREMENTS DRAFT:
E-commerce platform for small merchants. 50K products. Must scale to 100K concurrent users. 6-month launch. $200K annual budget.

STAKEHOLDER ROLES TO SIMULATE:
- Product Owner (launch first, iterate later)
- CFO (cost control)
- CISO/Security Lead (breach risk)
- SRE (operational stability)
- End User (merchant using the platform)
```

---

### The Prompt

```text
For each stakeholder role below, simulate their perspective on these requirements.

For each role, output:
1. Top 3 concerns they would raise (specific to their role)
2. Top 3 missing requirements they'd add
3. 1 hard question they'd ask the team
4. 1 risk they'd flag

Role personas and their primary incentives:

PRODUCT OWNER — Goal: Ship first, iterate fast
Concern: Schedule risk, scope creep, dependency blocking launch
Missing: Phased rollout plan? MVP scope clear? Tech debt tracked?

SECURITY LEAD — Goal: Zero breach incidents
Concern: Data exposure, access control, vendor security, audit trail
Missing: Threat model? Security testing plan? Vendor risk assessment?

SRE/PLATFORM LEAD — Goal: Operational stability, zero surprise failures
Concern: Scaling assumptions unvalidated, monitoring blind spots, runbook gaps
Missing: Capacity plan? Incident response plan? Monitoring/alerting strategy?

CFO — Goal: Cost predictability and ROI
Concern: Runaway infrastructure costs, licensing, hidden overhead
Missing: Cost model? Vendor lock-in analysis? TCO vs. build internally tradeoff?

COMPLIANCE OFFICER — Goal: Regulatory adherence
Concern: Data residency, retention, audit log sufficiency
Missing: Compliance checklist? Data flow mapping? Third-party risk assessment?

END USER (Merchant) — Goal: Simple, fast workflow
Concern: Onboarding friction, feature gaps, uptime during their peak hours
Missing: Bulk operations? API for automation? Uptime SLA?

For each role, explain their concern in a way that shows understanding of their constraints.
Example good concern: "SECURITY: We need to know who accessed customer data and when, but currently there's no audit logging mentioned"
Example weak concern: "SECURITY: Data needs to be secure" (generic, not actionable)

Do NOT invent solutions. Only identify concerns and missing requirements.

***
[PASTE YOUR REQUIREMENTS AND STAKEHOLDER ROLES HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Depth** | You want thorough coverage | Add: "For each stakeholder, also identify what success metric they'd use to measure the system (their KPI)" |
| **Focus** | You want priority-order | Add: "Rank stakeholder concerns by business impact (what could derail the project?)" |
| **Team context** | You have specific team dynamics | Add: "Assume [specific constraint] and ask: How does that change the SRE's concerns? The CFO's?" |
| **Scope** | You want to exclude certain roles | Add: "Only simulate Product Owner, Security, and SRE; skip others" |

---

### Example Output (Strong)

```
PRODUCT OWNER (Goal: Ship first, iterate fast)
Concerns:
1. "6 months is tight. Which requirements are truly launch-blocking? Which can be post-launch?"
2. "Dependency on merchant's ERP sync feels risky for launch—do we have manual fallback?"
3. "Search index corruption would block users. What's the recovery procedure?"

Missing requirements:
- MVP scope definition (e.g., "Day-1 features: catalog upload, search, basic wishlist. Post-launch: recommendations, analytics")
- Rollback plan (if merchants complain search is broken, can we roll back?)
- Tech debt budget (accept some shortcuts for launch?)

Hard question: "If we ship with single-region database and single-zone storage, what breaks first as we scale?"

Risk: Launch delay due to underestimated infrastructure work (auto-scaling, monitoring setup)

---

SECURITY LEAD (Goal: Zero breaches)
Concerns:
1. "Third-party ERP integrations via webhook—what if a merchant's ERP is compromised and sends malicious data?"
2. "Audit logging not mentioned. How will we know who accessed customer data and when?"
3. "No mention of customer data encryption at rest. Is GDPR compliance actually covered?"

Missing requirements:
- Threat model (document attack scenarios: data exfiltration, unauthorized admin access, webhook injection)
- Vendor risk assessment (ERP vendors: Are they SOC2 certified? What's their incident response SLA?)
- Data classification (which data is PII? Which is merchant-sensitive? Affects encryption + access control)

Hard question: "If a merchant uploads a catalog with SQL injection payloads or malware, what stops it?"

Risk: Breach during launch could halt the project and damage reputation

---

SRE LEAD (Goal: Operational stability)
Concerns:
1. "Scaling to 100K concurrent users with no capacity plan or load test. Don't know if design actually works."
2. "Merchants trigger mass-scale searches (flash mob effect at 9am each day). Needs safeguards."
3. "ERP webhook SLA not covered. If webhook provider is down, does our system degrade gracefully or fail hard?"

Missing requirements:
- Incident response runbook (if search is down, who's on-call? What's the playbook?)
- Monitoring thresholds (at what point do we auto-scale? At what point do we page the team?)
- Disaster recovery (if primary DB fails, can we failover to standby? How fast? How much data loss acceptable?)

Hard question: "You want 99.5% uptime. What's your acceptable RTO (recovery time) and RPO (data loss) for the database?"

Risk: Launch week outage due to scaling assumptions being wrong

---

CFO (Goal: Cost control)
Concerns:
1. "100K concurrent users on managed cloud services could run $5K+/month if not optimized. What's the cost model?"
2. "Third-party ERP integrations might have surprise licensing costs. Who pays for Stripe, auth providers, etc.?"
3. "Single-region deployment now, multi-region later? Refactoring later could double our infrastructure spend."

Missing requirements:
- Cost model by component (database, search, compute, bandwidth, managed services)
- TCO vs. build internally (why cloud? Why not on-prem in 6 months if we're budgeting $200K/year anyway?)
- Vendor lock-in risks (AWS pricing changes? We stuck?)

Hard question: "If scaling to 1M concurrent users becomes a goal next year, does your architecture choice today lock us into expensive refactoring?"

Risk: Cost overruns derail project post-launch
```

---

### Validation Checklist

- [ ] **Role-specific concerns:** Each stakeholder surfaces real concerns tied to their role/KPI (not generic concerns)
- [ ] **Different priorities:** Stakeholders disagree on some points (shows you've surfaced real tension)
- [ ] **Grounded in requirements:** All concerns reference something in the provided requirements or its implications
- [ ] **No invented solutions:** Output identifies problems, not architecture solutions
- [ ] **Hard questions answerable:** Clarifying questions require stakeholder input, not domain expertise

---

## ✓ Requirements → Clarifying Questions

### Use This When
You have requirements but know there are still gaps. Use this to generate a prioritized list of clarifying questions organized by business risk. Collect answers, then re-run ambiguity scan.

### When NOT to Use
Skip if your timeline allows full resolution before proceeding to architecture.

### Prerequisites
- Requirements draft (functional and non-functional)
- Stakeholders available to answer (or willing to try)
- Risk tolerance known (what's a showstopper? What can you compromise on?)

**Validation Rule:** If requirements are missing or too vague, output will say `[INPUT MISSING: Requirements]`. Gather a draft first—even a messy one. If requirements exist but are thin on constraints, output will have many [ASSUMED] markers — use those to guide what to clarify first.

---

### Input Example

```
REQUIREMENTS DRAFT:

Functional:
- Customers search products by keyword, category, price range
- Results page in < 1 second
- Merchants upload catalogs in CSV/XML
- Inventory updates in "near real-time"

Constraints Known:
- Team: 3 backend engineers
- Budget: $150K/year
- Availability: "Should be reliable"
- Deployment: AWS

Constraints Unknown:
- Concurrent user count
- Catalog size
- Merchant count
- Growth projections
```

---

### The Prompt

```text
Generate a prioritized list of clarifying questions that must be answered before architecture work begins.

Organize questions by business risk:

TIER 1: SHOWSTOPPERS
(Answer these first. If the answer changes, architecture changes completely.)

Example: "Do you have 10 concurrent users or 100K? This determines database choice."

TIER 2: CRITICAL PATH
(These affect design but not fundamental direction. Answer before design review.)

Example: "Is search result freshness 1 second? 1 minute? 1 hour? Drives caching strategy."

TIER 3: OPERATIONAL
(These affect ops complexity and costs. Answer before deployment planning.)

Example: "Can you afford 24/7 on-call? Or do you need managed services?"

TIER 4: NICE-TO-HAVE
(These can wait until post-launch iteration.)

Example: "Do you need analytics on search behavior? This affects data pipeline."

For each question:
1. State the question clearly (not leading, no assumed answer)
2. Explain why the answer matters (how it affects architecture)
3. Provide what happens if answer is unknown (what would you assume?)

Format:
[TIER X] [Category]: [Question]
Why it matters: [Architecture consequence]
If unknown, we'd assume: [Default assumption]
What that means: [Impact on budget/timeline/risk]

Do NOT ask for solutions or opinions. Ask for facts and constraints.

***
[PASTE YOUR REQUIREMENTS AND KNOWN/UNKNOWN CONSTRAINTS HERE]
```

---

### Customization Knobs

| Knob | Use When | How to Apply |
|------|----------|--------------|
| **Depth** | You want deep coverage | Add: "Also ask about team skills, prior architecture experience, and what mistakes to avoid" |
| **Risk-focused** | You're worried about specific areas | Add: "Prioritize questions about regulatory compliance and data security" |
| **Post-decision** | You've picked architecture; need operational details | Add: "Focus on TIER 3 (operational) questions only; skip TIER 1" |
| **Stakeholder-specific** | You want different questions per role | Add: "Generate 3 questions for Product Owner, 3 for SRE, 3 for CFO" |

---

### Example Output (Strong)

```
TIER 1: SHOWSTOPPERS

[Concurrency & Load]:
Question: "How many concurrent users must the system handle?"
Why: Search architecture, database type, caching strategy depend on this
  - < 100: Single database, basic indexing OK
  - 1K–10K: Need caching layer (Redis) and read replicas
  - 100K+: Need distributed search (Elasticsearch or similar)
If unknown, we'd assume: 5K concurrent (common for e-commerce startups)
Risk: Underestimate and system crashes at launch; overestimate and costs explode

[Catalog Size]:
Question: "How many products will be in the catalog at launch? Projected in 12 months?"
Why: Determines search technology and database schema
  - < 100K: SQL full-text search OK
  - 100K–1M: Need Elasticsearch or similar
  - 1M+: Need distributed search + caching
If unknown, we'd assume: 500K products (mid-market e-commerce)
Risk: Schema designed for 100K will require refactoring at 1M

[Real-Time Sync Definition]:
Question: "By 'near real-time,' do you mean: seconds? 5 minutes? 15 minutes? Or eventual consistency is OK?"
Why: Determines whether ERP sync uses webhooks, polling, or streaming
  - Seconds: Event streaming (expensive, complex)
  - 5–15 min: Webhook + retry logic (reasonable)
  - Eventual: Polling (cheapest, simplest)
If unknown, we'd assume: 15-minute sync window (cost/complexity balance)
Risk: Build for "seconds" when "15 minutes" acceptable; triple costs

[Data Retention]:
Question: "How long must customer wishlists, search history, order history be retained?"
Why: Affects database storage, archival strategy, compliance obligations
If unknown, we'd assume: Keep indefinitely (safest but expensive)
Risk: Underestimate and lose legal case; overestimate and incur unnecessary storage costs

TIER 2: CRITICAL PATH

[Peak Load Timing]:
Question: "When do you expect peak usage? (Time of day? Seasonal?)"
Why: Determines if we need auto-scaling or can get away with fixed capacity
If unknown: Assume constant load (simplifies ops; may overprovision)

[Search Tolerance]:
Question: "Is search result freshness < 1 second hard requirement, or approximate?"
Why: "Hard" means search index must be updated synchronously with catalog; "approximate" allows lag
If unknown: Assume hard requirement (safer but more expensive)

[Geographic Redundancy]:
Question: "Must the system survive entire AWS region failure? Or single-zone failure only?"
Why: Determines multi-region architecture vs. single-region with failover
If unknown: Assume single-region with AZ redundancy

TIER 3: OPERATIONAL

[Uptime SLA]:
Question: "When you say 'should be reliable,' do you mean 99% (2 hours downtime/month), 99.5%, or 99.9%?"
Why: Determines whether on-call coverage and automated failover are required
If unknown: Assume 99.5% (standard for business-critical systems)
Risk: Underestimate and miss incidents; overestimate and costs spiral

[On-Call Model]:
Question: "Can your team provide 24/7 on-call support? Or do you need managed services?"
Why: Affects whether you use AWS RDS/managed Elasticsearch or self-managed
If unknown: Assume managed services (shift ops burden to vendor)

[Disaster Recovery]:
Question: "If production database is completely lost, how much data loss is acceptable? How fast must recovery be?"
Why: RPO/RTO determines backup strategy, replication, standby infrastructure
If unknown: Assume 1-hour RPO, 1-hour RTO

TIER 4: NICE-TO-HAVE

[Analytics & Monitoring]:
Question: "Do you need analytics on user search behavior? A/B testing on search ranking?"
Why: Affects data pipeline, warehouse, and analytics infrastructure
If unknown: Assume basic monitoring only; skip advanced analytics post-launch

[Search Ranking Customization]:
Question: "Should each merchant be able to customize search ranking? (Promoted products, etc.)"
Why: Affects search system complexity and backend architecture
If unknown: Assume no customization; global ranking only
```

---

### Validation Checklist

- [ ] **Prioritized:** Questions grouped by showstopper/critical/operational/nice-to-have (not random order)
- [ ] **Architecture consequence stated:** For each question, output explains how the answer affects design
- [ ] **Concrete examples:** Questions include specific numbers or scenarios (not abstract)
- [ ] **No leading questions:** Questions don't assume an answer ("Shouldn't you use Kubernetes?" is leading)
- [ ] **Answerable:** Questions ask for facts/constraints, not opinions ("What's your favorite database?" ❌ → "What database experience does your team have?" ✓)

---

---

## How to Use

### For Initial Scoping

1. Start with **Requirements Ambiguity Scan** to surface obvious gaps in written requirements
2. Run **Requirements Deep Dive** to extract implicit NFRs and domain rules
3. Use **Stakeholder Perspective Simulation** to uncover hidden requirements from different roles

### For Clarification Loops

4. Generate **Requirements → Clarifying Questions** organized by risk category (security, performance, compliance, operational)
5. Collect answers from stakeholders
6. Re-run ambiguity scan and deep dive to identify new gaps

### Iteration Guidance

- **First pass:** Identify *what is missing*. Prioritize by risk (security > reliability > cost > usability)
- **Second pass:** Validate *what is stated*. Flag contradictions between functional and non-functional requirements
- **Final pass:** Extract *implied constraints*. Map to shared vocabulary in `/shared/constraints-vocabulary.md`

---

## Customization Guidance

### By Project Phase

- **Early Discovery:** Use full scope (ambiguity scan + deep dive + stakeholder simulation)
- **Active Development:** Use targeted clarification questions only (skip full deep dive)
- **Late-Stage Refinement:** Use stakeholder simulation focused on edge cases and exceptions

### By Audience

- **Cross-functional team:** Use stakeholder simulation to surface disagreement early
- **Single Product Owner:** Use ambiguity scan + clarifying questions
- **Regulated domain (finance, healthcare, etc.):** Add explicit compliance stakeholder perspective

### By Input Quality

- **Well-written functional specs:** Start with deep dive, skip ambiguity scan
- **Vague or conversational requirements:** Start with ambiguity scan, then deep dive
- **Mix of specs + verbal context:** Run stakeholder simulation first to surface implicit knowledge

---

## Quality Checks

### Before Using Output

- [ ] All **inferred content** is marked `[INFERRED]` — not presented as stated fact
- [ ] All **ambiguities** cite the specific requirement that triggered them (not vague)
- [ ] All **clarifying questions** are answerable (no yes-no questions alone; ask for specifics)
- [ ] All **stakeholder concerns** are grounded in stakeholder role, not invented

### Red Flags (Hallucination Indicators)

- ❌ Prompts list "all implicit NFRs" (hallucination risk — use `[INFERRED]` marker instead)
- ❌ Stakeholder personas provide solutions, not just concerns (out of scope; use for adversarial review instead)
- ❌ Clarifying questions reference technology or architecture (too specific; stay focused on *requirements*, not solutions)
- ❌ Output assumes team size, budget, or deployment environment that wasn't stated (mark as `[ASSUMED]`)

### Success Indicators

- ✅ Output highlights contradictions between requirements (not just gaps)
- ✅ Clarifying questions organize by *category* (performance, security, compliance, operational) — helps prioritize
- ✅ Stakeholder simulation includes security, SRE, and compliance perspectives (not just product/user perspective)
- ✅ No invented features or capabilities appear in the requirements extraction

---

## Next Steps

After clarification:
1. Compile answers to clarifying questions into a **constraints inventory**
2. Move to **02 — Architecture Exploration** with clear, complete requirements
3. Document all clarifications as assumptions in your architecture decision records

---

## Resources

- Shared constraints vocabulary: `/shared/constraints-vocabulary.md`
- Shared personas: `/shared/personas.md`
- Anti-slop checklist: `/shared/anti-slop-checklist.md` (use to validate output quality)

---

**Status:** Complete (4 prompt cards, validation rules, guardrails applied, anti-hallucination markers)
**Last Updated:** 2026-06-02 (Morpheus — Cross-Section Remediation)
