# Document Scope Reference

Guidelines for what belongs in each architecture document type and what should be excluded.

---

## Purpose

Architecture documents serve different purposes. Keeping scope clear prevents clutter, confusion, and contradictions. This guide defines what belongs in each document type.

---

## Document Type Scopes

### 1. Architecture Decision Record (ADR)

**Purpose:** Answer "WHY did we choose this architecture?" — not "HOW do we implement it?"

**Scope:** In (Strategic-level decisions)
- ✓ Why we chose technology X over Y
- ✓ Constraints that motivated the decision
- ✓ Options considered + tradeoffs
- ✓ Consequences (what becomes easier/harder)
- ✓ Related decisions (ADR dependencies)

**Scope:** Out (Implementation details)
- ✗ Specific library versions (Node.js 16.x vs. 18.x)
- ✗ Configuration parameters (max_connections = 500)
- ✗ SQL query examples
- ✗ Code patterns (async/await syntax)
- ✗ Vendor-specific commands (aws cli parameters)

**Length:** 1–3 pages per decision

**Audience:** Architects, team leads, future decision-makers

**Example of scope creep (❌) and fix (✓):**

❌ **Too much implementation detail:**
```
ADR-001: Use PostgreSQL
Decision: PostgreSQL for strong consistency
Implementation: Use pg_stat_statements view to monitor queries.
Create indexes on customer_id, created_at. Set max_connections = 500.
Use connection pooling via PgBouncer.
```

✓ **Correct scope:**
```
ADR-001: Use PostgreSQL for strong consistency
Decision: PostgreSQL provides ACID transactions (required for order consistency)
Consequences: (1) Requires database tuning expertise (address in hiring), (2) Monitoring infrastructure needed (address in ops spec)
Related: See Ops Spec for database configuration + monitoring
```

---

### 2. Architecture Specification

**Purpose:** Answer "WHAT are the components?" and "HOW do they talk to each other?"

**Scope:** In (Component-level design)
- ✓ Component responsibilities (one sentence each)
- ✓ Component interfaces (APIs, protocols, formats)
- ✓ Technology choices with rationale
- ✓ Data flows (end-to-end requests)
- ✓ Scalability approach (horizontal? vertical? caching?)
- ✓ Failure modes + recovery

**Scope:** Out (Implementation details or business processes)
- ✗ Specific SQL queries or index definitions
- ✗ Library selection (Express vs. Fastify)
- ✗ Code examples
- ✗ Deployment commands
- ✗ Business logic (how inventory manager approves stock)
- ✗ Pricing or revenue models

**Length:** 5–10 pages per system

**Audience:** Architects, tech leads, implementers

**Example of scope creep (❌) and fix (✓):**

❌ **Too much implementation:**
```
Order Service component:
- Technology: Node.js 18.x with Express 4.18.2
- Dependencies: uuid, joi, lodash
- Folder structure: /src/routes, /src/controllers, /src/models
- Sample code:
  const order = await Order.findById(orderId);
  if (!order) throw new NotFoundError();
```

✓ **Correct scope:**
```
Order Service component:
- Responsibility: Accepts customer orders, validates constraints, coordinates fulfillment
- Technology: Node.js + REST API (chosen because team has Node.js expertise)
- Interfaces: POST /orders (input: items array, address, payment token; output: order ID)
- Dependencies: PostgreSQL (orders), Inventory Service (reserve stock), Payment Service (authorize)
- Scalability: Stateless; scales horizontally; bottleneck at database write throughput (100K/sec)
See Implementation Guide for code structure and library specifics.
```

❌ **Too much business logic:**
```
Order Service also enforces:
- Inventory manager approval for orders >$10K
- Fraud detection for repeat customers in high-risk regions
- Dynamic pricing based on inventory levels
```

✓ **Correct scope:**
```
Order Service enforces:
- Inventory constraints (validates stock available before order accepted)
- Business rules delegated to separate services:
  - Approval workflow: Handled by Approval Service (separate system)
  - Fraud detection: Handled by Risk Service (separate system)
  - Pricing: Handled by Pricing Service (separate system)
```

---

### 3. Process Flow Document

**Purpose:** Answer "WHAT IS THE STEP-BY-STEP SEQUENCE?" including decision points and error handling

**Scope:** In (Operational/technical sequences)
- ✓ Steps in order (numbered)
- ✓ Who/what performs each step (actors)
- ✓ Inputs and outputs of each step
- ✓ Decision points (IF/THEN branches)
- ✓ Error handling (explicit error paths)
- ✓ Timeouts and retries
- ✓ Rollback procedures

**Scope:** Out (Business process or implementation)
- ✗ Business rules (when inventory manager approves)
- ✗ Why business makes decisions (strategy, policy)
- ✗ Implementation details (specific SQL queries, API endpoint URLs)
- ✗ User interface mockups
- ✗ Reporting or analytics

**Length:** 2–5 pages per process

**Audience:** Operations, on-call engineers, QA

**Example of scope creep (❌) and fix (✓):**

❌ **Too much business process:**
```
Step 3: Inventory Manager Reviews Inventory Levels
- Manager checks current stock in ERP system
- If stock low, manager consults with supplier contracts
- Manager negotiates pricing and timing
- Manager approves purchase order
```

✓ **Correct scope (technical flow):**
```
Step 3: Inventory Service Checks Stock Levels
- [ORDER_SERVICE] calls GET /inventory/{sku}
- [INVENTORY_SERVICE] queries PostgreSQL
- Returns: {available: N, reserved: M}
- [ORDER_SERVICE] reserves inventory (separate transaction)
```

(Keep business process in separate Business Process document, not technical flow)

---

### 4. Risk Register

**Purpose:** Answer "WHAT COULD GO WRONG?" with probability, impact, and mitigation

**Scope:** In (Risks specific to this architecture)
- ✓ What failure could occur (specific to YOUR architecture)
- ✓ Probability (grounded in facts or architecture)
- ✓ Impact (business, technical, operational)
- ✓ Current mitigations (what you're already doing)
- ✓ Recommended mitigations (cost, timeline, effort)
- ✓ Contingency (if it happens, what's the response?)
- ✓ Monitoring (how to detect)

**Scope:** Out (General risks or compliance boilerplate)
- ✗ Generic risks ("system might fail" without specifics)
- ✗ Risks unrelated to architecture ("key person leaves" — ops risk, not architecture risk)
- ✗ Generic compliance risks (copy/pasted from templates)
- ✗ Speculative risks (invented scenarios without basis)

**Length:** 0.5–1 page per risk

**Audience:** CTO, SRE, Project Manager

**Example of scope creep (❌) and fix (✓):**

❌ **Too generic:**
```
RISK: System fails
Impact: Customers can't access system
Mitigation: Improve reliability
```

✓ **Specific to architecture:**
```
RISK: Payment Gateway outage blocks all orders
Trigger: Payment Gateway returns 5xx errors for >1 minute
Probability: Medium (Stripe SLA 99.99% = 4 hours/year downtime)
Impact: Revenue: $5K/hour lost; Operational: Manual order processing required
Current Mitigation: Timeout 5 seconds + retry 2x
Recommended: Persist queued orders to DB; async retry every 2 minutes for 1 hour
Contingency: If >1 hour: Support team manually processes orders via Stripe dashboard (1 order/minute)
Monitoring: Alert if error rate >5% for 1 minute
```

---

### 5. Assumption Register

**Purpose:** Answer "WHAT ARE WE ASSUMING?" with validation plans and fallbacks

**Scope:** In (Assumptions underlying architecture decisions)
- ✓ Assumption statement (specific claim)
- ✓ Why it matters (which decision depends on it?)
- ✓ Confidence level (1–5)
- ✓ Validation plan (how to test)
- ✓ Fallback (if assumption is wrong, plan B?)

**Scope:** Out (Generic project risks or business assumptions)
- ✗ "Team will be motivated" (team psychology, not architecture)
- ✗ "Market will grow" (market risk, not architecture)
- ✗ "Customer will pay" (business risk, not architecture)
- ✗ Generic operational assumptions (budget exists, management support exists)

**Length:** 0.5–1 page per assumption

**Audience:** Architects, Tech Leads

**Example of scope creep (❌) and fix (✓):**

❌ **Too broad (not architecture-specific):**
```
ASSUMPTION: Team will stay motivated throughout 12-month project
ASSUMPTION: Customer demand will exceed sales projections
ASSUMPTION: Hiring will be successful
```

✓ **Architecture-specific:**
```
ASSUMPTION: Team can operationalize Kubernetes in 3 months (affects: deployment architecture choice)
ASSUMPTION: PostgreSQL will scale to 100K writes/sec (affects: database architecture choice)
ASSUMPTION: Payment Gateway latency <200ms p99 (affects: async-vs-sync architecture choice)
```

---

### 6. Traceability Matrix

**Purpose:** Answer "DO WE ADDRESS EVERY REQUIREMENT?" — map requirements → components → risks

**Scope:** In (Requirement coverage)
- ✓ Each requirement mapped to component(s)
- ✓ Each component mapped to requirement(s)
- ✓ Each risk mapped to mitigation component(s)
- ✓ Gaps identified (requirements with no component)
- ✓ Orphaned components (components without requirement)

**Scope:** Out (Implementation or testing details)
- ✗ Test cases or test coverage
- ✗ Implementation tasks or sprints
- ✗ Personnel assignments
- ✗ Budget/timeline (separate tracking)

**Length:** 1–2 pages (table format)

**Audience:** Project Manager, QA, Architects

**Example of scope creep (❌) and fix (✓):**

❌ **Too much implementation detail:**
```
REQ-001: Accept orders
Mapped to: Order Service
Implementation task: [Epic-42] Add /orders endpoint
Sprint: Q2 Sprint 3
Assigned to: Engineer A, Engineer B
Estimated effort: 8 story points
Test cases: TestOrderValidation, TestInventoryCheck, ...
```

✓ **Correct scope:**
```
REQ-001: Accept orders
Mapped to: Order Service (component)
ADR: ADR-002 (component architecture decision)
Risk mitigated: RISK-005 (order processing delays)
Status: ✓ Addressed
```

---

### 7. Operational Runbook

**Purpose:** Answer "HOW DO WE RUN THIS OPERATIONALLY?" — procedures for deployment, monitoring, incident response

**Scope:** In (Operational procedures)
- ✓ Step-by-step deployment procedures
- ✓ Pre-flight checks
- ✓ Rollback procedures
- ✓ Critical alerts + response procedures
- ✓ Common failure scenarios + fixes
- ✓ Escalation procedures
- ✓ Monitoring dashboards + metrics
- ✓ On-call handoff template

**Scope:** Out (Architecture rationale or business logic)
- ✗ Why we chose this architecture (belongs in ADR)
- ✗ Design decisions or tradeoffs (belongs in spec)
- ✗ Business logic (belongs in business docs)
- ✗ Implementation code examples (belongs in code comments)

**Length:** Varies; 1 page per procedure minimum

**Audience:** On-call engineers, SRE, DevOps

**Example of scope creep (❌) and fix (✓):**

❌ **Mixing architecture + operations:**
```
Deployment Runbook: Order Service
We chose microservices to enable independent deployments (architecture rationale).
The Order Service is a Node.js application that ...
Code is structured as: /src/routes, /src/controllers, /src/models
Dependencies: Express, lodash, uuid
```

✓ **Operations-only:**
```
Deployment: Order Service
Pre-deployment: (1) Run test suite locally. (2) Check staging health. (3) Verify database backups completed.
Deployment: (1) Build Docker image. (2) Push to ECR. (3) Update Kubernetes deployment.
Verification: (1) Smoke tests pass. (2) Error rate <1% for 5 minutes. (3) Latency p99 <500ms.
Rollback: If error rate spikes >5% or latency p99 >2s: kubectl rollout undo deployment/order-service
See Architecture Spec for component overview
```

---

## Scope Boundary Cheat Sheet

| What You're Wondering | Architecture Doc | Implementation Doc |
|---------------------|--------------------|-------------------|
| "Why this tech?" | ✓ ADR | ✗ Code |
| "How do these components talk?" | ✓ Spec | ✗ Code |
| "What are the step-by-step operations?" | ✓ Process Flow | ✗ Code |
| "What could go wrong?" | ✓ Risk Register | ✗ Incident Reports |
| "What are we assuming?" | ✓ Assumptions | ✗ Meeting notes |
| "Does this cover all requirements?" | ✓ Traceability | ✗ Tickets/Sprints |
| "How do we deploy?" | ✓ Runbook | ✗ Shell scripts |
| "How do we implement the database?" | ✗ Not architecture | ✓ Implementation guide |
| "What code libraries should we use?" | ✗ Not architecture | ✓ Implementation guide |
| "What config values?" | ✗ Not architecture | ✓ Operations runbook |

---

## Red Flags: Out-of-Scope Content

| Red Flag | Likely Out of Scope | Move To |
|----------|-------------------|----------|
| Specific code examples | Everywhere except Runbook | Implementation guide or code comments |
| Library/version names | Spec, ADR | Implementation guide |
| Config values (max_connections=500) | Spec, ADR, Risk | Operations runbook |
| SQL queries or database schema | Spec, ADR | Database design doc |
| Business logic details | Process Flow, Spec | Business process doc |
| Personnel/scheduling | Any architecture doc | Project plan / tracking |
| Budget breakdown | Risk, Spec | Financial tracking |
| Sprint planning | Any architecture doc | Project backlog |
| Testing details | Spec, ADR | Test plan document |

---

## Mixed-Scope Documents: How to Handle

**Sometimes a document needs to reference multiple scopes. Strategy:**

1. **Keep primary scope focused** — If it's an ADR, focus on WHY; reference (not include) HOW
2. **Use "See Also" sections** — Link to related documents
3. **Separate appendices** — Detailed implementation examples in appendix, not main body

**Example (Good):**
```
ADR-001: Use PostgreSQL

DECISION: PostgreSQL for strong consistency
RATIONALE: [architecture reasoning]
CONSEQUENCES: [affects development + ops]

See Also:
- Architecture Spec for PostgreSQL component details
- Operations Runbook for database monitoring + backup procedures
- Implementation Guide for schema design + optimization

Appendix: Example connection string (for reference only)
```

**Example (Bad — mixed scopes):**
```
ADR-001: Use PostgreSQL

DECISION: PostgreSQL for strong consistency
[Correct so far]

IMPLEMENTATION:
CREATE TABLE orders (...); -- WRONG: belongs in implementation doc
CREATE INDEX idx_customer_orders ON orders(customer_id); -- WRONG: ops doc
Set max_connections = 500; -- WRONG: ops doc
Use connection pool PgBouncer; -- WRONG: implementation doc
```

---

## Scope Approval Checklist

**Before finalizing any architecture document:**

- [ ] Document type is named (ADR, Spec, Runbook, etc.)
- [ ] Scope is stated (what this doc covers; what it doesn't)
- [ ] Content stays within stated scope (no scope creep)
- [ ] Out-of-scope items reference other docs (with "See Also" links)
- [ ] Audience is clear (who should read this?)
- [ ] Audience gets what they need (architects, implementers, or ops?)
- [ ] Redundancy minimized (no duplication with other docs)
- [ ] References are complete (every external reference has a link or clear location)
