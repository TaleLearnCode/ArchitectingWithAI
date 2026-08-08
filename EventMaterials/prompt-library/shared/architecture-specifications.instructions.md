# Architecture Specifications Instructions

Template for documenting component-level architecture: responsibilities, interfaces, technology choices, rationale, and data flows.

---

## Purpose

An Architecture Specification defines **what each component does**, **how they talk to each other**, and **why you chose each technology**. Specs are the bridge between architectural decisions (ADRs) and implementation details.

---

## Template Structure

### 1. Overview

**System Purpose:** One sentence. What does the system do?  
**Key Design Principles:** 2-3 principles that guide component design (e.g., "Loose coupling between order intake and fulfillment"; "Strong consistency for financial transactions")  
**Scope:** What's included? What's out of scope?

**Example:**
```
System Purpose: Process e-commerce orders, manage inventory, and coordinate fulfillment.

Key Design Principles:
- Order intake decouples from fulfillment (event-driven)
- Inventory is strongly consistent across all services
- Payment data never stored in logs or search systems

Scope: Covers order creation through fulfillment handoff. Does NOT include shipping carrier integration or customer returns (separate systems).
```

### 2. Component Diagram

**ASCII or text diagram showing connections:**

```
┌──────────────────────────────────────────────────────────┐
│                        Customer                          │
└────────────────────────┬─────────────────────────────────┘
                         │ HTTPS
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    API Gateway                           │
│       (rate limiting, auth, request routing)             │
└─┬──────────────────────────────────────────────────────┬─┘
  │                                                         │
  │ REST                                                    │ REST
  ▼                                                         ▼
┌─────────────────────┐                        ┌──────────────────────┐
│   Order Service     │────────────────────────│ Inventory Service    │
│ (business logic)    │        gRPC            │ (stock management)   │
└──┬──────────────────┘                        └──────────────────────┘
   │
   │ Publish OrderCreated
   ▼
┌──────────────────────────┐
│    RabbitMQ Message      │
│      Queue (Events)      │
└──┬───────────────────────┘
   │ Subscribe
   ▼
┌──────────────────────────┐
│ Fulfillment Service      │
│ (async order processing) │
└──────────────────────────┘
```

### 3. Component Details

For **each component**, document:

#### Component Name
- **Responsibility:** One sentence. What does this component do?
- **Owned by:** Team or group responsible (e.g., "Platform Team")
- **Running on:** Infrastructure (e.g., "Kubernetes, 3 replicas"; "AWS App Runner, auto-scaling 1–20")

#### Interfaces (APIs & Protocols)

**Incoming (synchronous):**
```
Endpoint: POST /orders
Protocol: REST/HTTPS
Input: { items: [{ id, quantity }], address, paymentToken }
Output: { orderId, status: "accepted", confirmationUrl }
Latency SLA: <500ms p99
```

**Incoming (asynchronous):**
```
Event: InventoryReserved
Broker: RabbitMQ
Input: { orderId, items: [{sku, quantity}], reservationId }
Processing: At-least-once delivery; idempotent by orderId
```

**Outgoing (calls other services):**
```
Endpoint: GET /inventory/{sku}
Target Service: Inventory Service
Latency expectation: <100ms p99
Timeout: 2 seconds; circuit breaker after 3 consecutive failures
Fallback: Use cached inventory (5-minute TTL)
```

#### Dependencies (What This Component Needs)

**Internal (other components):**
- Inventory Service: `GET /inventory/{sku}` for stock checks
- Payment Service: `POST /authorize` for payment processing

**External (databases, caches, third-party APIs):**
- PostgreSQL (orders table): Write-heavy, 1M rows, indexed on customer_id + created_at
- Redis (session cache): Caching cart state; 100MB TTL 1 hour
- Stripe API: Payment authorization; SLA 99.99%

**Infrastructure:**
- Message broker: RabbitMQ
- Logging: Elasticsearch
- Monitoring: Prometheus + Grafana

#### Technology Choice & Rationale

For **each technology decision**, ground it in your stated **constraints** and **requirements**:

**Example:**

| Decision | Technology | Rationale | Constraint It Satisfies |
|----------|-----------|-----------|------------------------|
| Language | Node.js + Express | Team has 4 Node.js engineers; rapidly iterate on business logic | Team skill constraint; time-to-market |
| Database | PostgreSQL | ACID transactions critical for orders; team familiar with SQL | Data consistency requirement; team skill |
| Cache | Redis | Sub-10ms lookups; session state ephemeral; well-understood by team | Latency SLA (<500ms p99); operational maturity |
| Message Broker | RabbitMQ | Decouples order intake from fulfillment; dead-letter queues for error handling | Scale requirement; operational maturity |
| Deployment | Docker on Kubernetes | Stateless services scale horizontally; managed service reduces ops load | Scale & operational maturity; team learning curve 2 months |

**❌ Weak rationale:** "Node.js is modern and scalable."  
**✓ Strong rationale:** "Node.js chosen because 4 of 8 engineers know it; supports rapid iteration on business logic (req: 6-month TTM); non-blocking I/O handles concurrent requests efficiently (req: 100K concurrent users)."

#### Scalability Approach

- **Horizontal scaling:** Stateless services; add replicas. Load balancer distributes traffic.
- **Vertical scaling:** Database indexing; caching layer; query optimization.
- **Data partitioning:** Orders by customer_id (if >1B rows); can shard by customer_id ranges.
- **Rate limiting:** API Gateway enforces 100 req/sec per customer; queues overflow to message broker.
- **Bottleneck:** Database write throughput at >10K orders/sec; mitigated by async fulfillment (decouples intake from processing).

#### Failure Modes & Recovery

| Failure | Impact | Detection | Recovery | RTO |
|---------|--------|-----------|----------|-----|
| Service crashes | Customers get 5xx; queued orders stuck | Kubernetes restarts container; PagerDuty alert if restart loop | Auto-restart; if persistent, drain connections and debug | 5–10 sec (k8s restart) |
| Database connection lost | All writes fail; reads timeout | Connection pool exhausted; queries pile up; alert on connection count | Retry logic; circuit breaker; manual failover to read replica | 30–60 sec (detect + failover) |
| Message broker down | Events not published; fulfillment stalls | Alert on broker health check; consumer lag increases | Restart broker; replay messages from persistent log | 1–2 min (restart + replay) |
| Inventory check timeout | Order processing stalls; latency SLA violated | Latency p99 > 500ms; circuit breaker triggers | Use cached inventory (5-min TTL); flag order for async verification | 2 sec (fallback cache) |

#### Consistency Model

- **Order Service & Inventory Service:** Eventual consistency. Order reserved in Order Service; inventory reserved in Inventory Service via async message. If Inventory fails, Order Service has compensation.
- **Payment Service & Order Service:** Strong consistency. Payment must succeed before order created (synchronous call with timeout).
- **Order Service & Fulfillment:** Eventual consistency. Order published as event; fulfillment consumes independently. If fulfillment crashes, order persists; fulfillment retries.

### 4. Data Flow

**Trace one critical user request end-to-end:**

```
User submits order:
  1. API Gateway receives POST /orders
  2. Order Service: Validate order (items, quantities, address)
  3. Inventory Service: Reserve items (gRPC call; timeout 2s)
  4. Payment Service: Authorize payment with Stripe (REST; timeout 5s)
  5. Order Service: Write order record to PostgreSQL
  6. Order Service: Publish OrderCreated event to RabbitMQ
  7. Email Service: (async) Send confirmation email (SQS; non-blocking)
  8. Fulfillment Service: (async) Subscribe to OrderCreated; pick items
  
  Total latency (p99): Order Service (50ms) + Inventory RPC (100ms) + Payment API (200ms) + DB write (50ms) + event publish (10ms) = 410ms (within 500ms SLA)
```

### 5. External Dependencies

List every third-party or external system:

| Dependency | Type | SLA | Fallback | Impact if Down |
|-----------|------|-----|----------|----------------|
| Stripe Payment API | SaaS | 99.99% | Manual authorization queue (process within 1 hour) | Checkout blocked; revenue impact; customer emails |
| RabbitMQ | Broker | 99.9% (self-managed) | In-memory queue (if broker down <5min); replay from database after restart | Fulfillment delayed; backlog clears when broker restarts |
| Elasticsearch | Logging | 99% (best-effort) | Fallback to file storage | Search logs temporarily unavailable; operational debugging slower |
| AWS (Compute, RDS, S3) | Infrastructure | 99.95% | Multi-AZ RDS failover automatic; S3 has redundancy | Entire system down; RTO ~1 minute (RDS failover + app restart) |

### 6. Security & Data Governance

- **Authentication:** OAuth 2.0 (customers); mTLS (service-to-service)
- **Authorization:** Role-based access control (RBAC); Order Service enforces: customers can only access their own orders
- **Data Encryption:** TLS for all transport; PostgreSQL encryption at rest; payment data never logged (PCI-DSS)
- **Compliance:** GDPR data residency (EU data stays in EU region); audit logs immutable (append-only)
- **Secrets:** Stripe API keys stored in AWS Secrets Manager; rotated quarterly

### 7. Monitoring & Observability

**Metrics to track per component:**

| Metric | Target | Alert Threshold | Owner |
|--------|--------|-----------------|-------|
| API latency (p99) | <500ms | >1s for 2 min | Platform Team |
| Order throughput | 1K–10K orders/min | <500 orders/min (drop >50%) | Platform Team |
| Payment success rate | >99% | <95% success | Payment Team |
| Inventory consistency lag | <5 sec | >30 sec | Inventory Team |
| Database query latency (p99) | <100ms | >500ms | DBA / Platform |

**Logs to capture:**

```
order_created: { orderId, timestamp, customerId, itemCount, totalPrice }
order_failed: { orderId, timestamp, reason, failureType, retryable }
payment_authorized: { orderId, amount, authorization_id, timestamp }
payment_failed: { orderId, amount, reason, timestamp }
inventory_reserved: { orderId, items, reservationId, timestamp }
inventory_failed: { orderId, reason, timestamp }
```

**Distributed tracing:** Use correlation IDs; trace order from API Gateway → Order Service → Inventory → Payment → Database.

### 8. Deployment & Release

- **Deployment model:** Docker containers on Kubernetes; namespaces per environment (dev, staging, prod)
- **Release strategy:** Blue-green deployments; feature flags for gradual rollout; automatic rollback if error rate > 5%
- **Backward compatibility:** New API versions coexist with old versions (e.g., /v1/orders, /v2/orders)
- **Database migrations:** Backward-compatible schema changes; separate deployment from application code

### 9. Related ADRs

Link to decisions this specification depends on:
- **ADR-001:** Use event-driven architecture (justifies async messaging choice)
- **ADR-003:** PostgreSQL for strong consistency (justifies DB choice)
- **ADR-005:** OAuth 2.0 for authentication (justifies auth model)

---

## Quality Checklist

- [ ] Each component maps to a requirement or architectural decision
- [ ] Technology choices have explicit rationale tied to constraints
- [ ] Data flows are traceable end-to-end
- [ ] Failure modes are realistic; not invented
- [ ] Monitoring signals are specific and actionable (not "monitor for issues")
- [ ] External dependencies documented with SLAs and fallbacks
- [ ] Scalability limits identified (where does bottleneck appear?)
- [ ] Consistency model explicit (not assumed)
- [ ] Security & compliance concerns addressed
- [ ] Related ADRs referenced

---

## Anti-Patterns

❌ **Generic tech rationale:** "PostgreSQL is reliable and widely used."  
✓ **Specific rationale:** "PostgreSQL chosen for ACID transactions (required for order consistency) and team SQL expertise (4 of 8 engineers); NoSQL alternatives considered but require eventual consistency, which violates financial transaction requirements."

❌ **Missing failure modes:** Assumes components always work  
✓ **Complete failure modes:** Specifies what happens when Stripe times out, DB connection lost, inventory service crashes

❌ **Vague monitoring:** "Monitor for issues"  
✓ **Specific monitoring:** "Alert if payment success rate < 95% for 5 minutes; trigger on-call escalation"

---

## Example: Complete Component Spec

**Component:** Order Service  
**Responsibility:** Accepts customer orders, validates constraints, coordinates with Inventory & Payment services, publishes OrderCreated events.  
**Interfaces:** REST /orders (sync), RabbitMQ OrderCreated (async), gRPC to Inventory Service  
**Dependencies:** PostgreSQL, RabbitMQ, Inventory Service, Payment Service  
**Technology:** Node.js + Express (team skill), PostgreSQL (consistency), RabbitMQ (decoupling)  
**Scalability:** Stateless; horizontal scaling via Kubernetes replicas (1–20 based on load); database write bottleneck at 10K+ orders/sec (mitigated by async fulfillment)  
**Failure:** If Inventory times out, use cache + flag for verification; if Payment fails, compensate by releasing inventory.  
**Monitoring:** API latency p99 <500ms; order throughput >1K/min; payment success >99%.
