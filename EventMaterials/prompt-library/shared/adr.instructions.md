# ADR (Architecture Decision Record) Instructions

Instructions and field template for generating and documenting architecture decisions with rationale, alternatives, and consequences.

---

## Purpose

An ADR captures **why** an architecture decision was made, not just **what** was decided. It serves as a decision log for future teams, stakeholders, and architects.

---

## Template Fields

### 1. Title
**One sentence.** State the decision in past tense.  
Example: "Use event-driven architecture for order processing"

### 2. Status
One of:
- `Accepted` — Decision implemented or approved for implementation
- `Pending` — Decision proposed, awaiting review/approval
- `Rejected` — Decision considered but not chosen
- `Superseded` — Decision was valid but replaced by later ADR

### 3. Context
**1–3 paragraphs.** Answer:
- What problem does this decision solve?
- What constraints apply? (team size, budget, SLA, regulatory, time-to-market)
- Why does this decision matter?
- What was the trigger for making this decision?

**Example:**  
*"Our order volume during flash sales spikes 10x, causing the request-reply API to queue incoming orders. Current monolith processes orders sequentially, creating 2-minute backlogs during peaks. With SLA of 99.5% uptime and team of 8 engineers, we need loose coupling between order intake and fulfillment. Budget is $150K/year; we have 6 months to implement."*

### 4. Decision
**One sentence.** State which option was chosen and why it fits the constraints.

**Example:**  
*"We will use event-driven architecture with RabbitMQ as the message broker. This decouples order intake from fulfillment, allowing the API to remain responsive during spikes while fulfillment processes scale independently."*

### 5. Options Considered
List **at least 2 alternatives.** For each option, provide:
- **Name/Short Description**
- **How it works** (1–2 sentences)
- **Why we rejected it** (or chose it)
- **Tradeoff vs. chosen option** (specific to your constraints)

**Example:**

| Option | Description | Fit to Constraints | Tradeoff |
|--------|-------------|-------------------|----------|
| **Monolithic Request-Reply** | Single Node.js service, requests queued in memory | Fits team (simple), violates SLA (2-min backlog during peaks) | Simpler ops, but fails under load; requires rewrite later |
| **Event-Driven (Chosen)** | RabbitMQ decouples order intake from fulfillment | Fits team (learnable in 2 months), fits SLA (API stays responsive), fits budget (~$500/yr RabbitMQ) | Requires team learning message patterns; adds operational complexity |
| **Microservices with API Gateway** | Separate Order, Fulfillment, Payment services | Fits SLA, but team needs 3+ months learning; exceeds 6-month runway | Over-engineered for current scale; unnecessary complexity |

### 6. Pros & Cons

**Pros (specific to your constraints):**
- Decouples intake from fulfillment: API responds within 50ms even if fulfillment is backlogged.
- Scales fulfillment independently: Add more workers without impacting API tier.
- Team can learn message patterns in 2 months: aligns with our runway.
- Cost: ~$500/yr for RabbitMQ (managed service); fits budget.

**Cons (and whether they're acceptable):**
- Adds operational complexity: Team must learn message ordering, dead-letter queues, retry logic. **MITIGATED: Schedule 1-week training sprint.**
- Requires monitoring for message lag: Need alerts on queue depth and processing latency. **ACCEPTABLE: Part of SRE maturation plan.**
- Debugging distributed failures harder than monolith. **MITIGATED: Add correlation IDs to all messages; document runbook in week 2.**

### 7. Consequences

**What becomes easier?**
- Scaling fulfillment independently (add workers without redeploying API).
- Running API replicas for HA without cascading failures.
- A/B testing fulfillment algorithms (swap implementations without API downtime).

**What becomes harder?**
- Distributed debugging (must understand async message flow).
- Testing (need to test message ordering, replay, and failure scenarios).
- On-call incident response (more moving parts; incident time increases ~10% initially, then improves).

**What operational practices must the team adopt?**
- Message ordering monitoring: Alert if messages arrive out of sequence.
- Dead-letter queue review: Weekly scan of poison messages.
- Runbook for message replay: If we need to reprocess orders, how do we do it safely?

### 8. Validation

**How will we know this decision was correct?**
- SLA metric: API response time remains <50ms p99 even during 10x load spikes.
- Operational metric: Fulfillment queue depth recovers to <100 messages within 2 minutes of spike.
- Team metric: On-call incidents attributed to message ordering drop to zero after first 3 months.

### 9. Review & Approval

- **Proposed by:** [Name]
- **Reviewed by:** [Names]
- **Approved by:** [Authority, e.g., CTO, Architecture Committee]
- **Decision date:** [YYYY-MM-DD]

### 10. Related Decisions

Link to ADRs this decision depends on or affects:
- **Depends on:** ADR-003 "Use PostgreSQL for strong consistency"
- **Affects:** ADR-004 "Implement monitoring for message lag" (needs to be created)
- **Superseded by:** [If later replaced, reference the new ADR]

---

## Quality Checklist

- [ ] Context explains the constraint(s) that motivated this decision
- [ ] Decision is grounded in constraints, not generic best practices
- [ ] Each option is tied to a specific tradeoff (not "better" or "worse" without context)
- [ ] Consequences are testable and measurable
- [ ] Consequences include both operational and team learning impacts
- [ ] No invented technologies or capabilities (only reference what's achievable with stated resources)
- [ ] If a consequence requires new tooling, it's flagged as [LEARNING_REQUIRED]

---

## Anti-Patterns (What to Avoid)

❌ **Generic reasoning:** "Microservices are scalable and modern."  
✓ **Specific reasoning:** "Event-driven decouples intake from fulfillment, keeping the API responsive during 10x spikes with our current team of 8."

❌ **Ungrounded options:** Listing options that violate constraints (e.g., "Kubernetes" when team has zero container experience and 6-month deadline).  
✓ **Grounded options:** Each option explicitly states FITS/VIOLATES/REQUIRES VALIDATION for all constraints.

❌ **Vague consequences:** "Improved scalability" or "better separation of concerns."  
✓ **Specific consequences:** "API response time remains <50ms p99 during spikes; team must learn message patterns (1-week sprint included in timeline)."

---

## Example: Complete ADR

**Title:** Use event-driven architecture for order processing  
**Status:** Accepted  
**Context:** Order volume spikes 10x during flash sales, causing monolith to queue orders for 2 minutes. SLA is 99.5%; team is 8 engineers; 6-month runway; $150K/yr budget.  
**Decision:** Event-driven with RabbitMQ. Decouples intake from fulfillment, allowing API to stay responsive while fulfillment scales independently.  
**Options Considered:**
| Option | Fit | Tradeoff |
|--------|-----|----------|
| Monolithic request-reply | Fits team, VIOLATES SLA (2-min backlog under load) | Simple ops, fails under spike |
| **Event-driven (chosen)** | Fits team, SLA, budget | Requires learning message patterns |
| Microservices + API Gateway | Fits SLA, VIOLATES timeline (6 months too long) | Over-engineered, unnecessary complexity |

**Pros:** Decouples tiers (API <50ms p99 even if fulfillment backlogged). Scales fulfillment independently. Team can learn in 2 months. Cost fits budget (~$500/yr).  
**Cons:** Adds operational complexity [LEARNING_REQUIRED: 1-week message pattern training]. Debugging harder [MITIGATED: correlation IDs, runbooks].  
**Consequences:** API becomes more reliable under spikes. Team adopts message queue discipline. Dead-letter queue monitoring becomes operational requirement.  
**Validation:** SLA maintained during spikes. Message ordering never violates. On-call incidents drop to zero after 3 months.  
**Related:** Depends on ADR-003 (PostgreSQL). Affects ADR-004 (monitoring).
