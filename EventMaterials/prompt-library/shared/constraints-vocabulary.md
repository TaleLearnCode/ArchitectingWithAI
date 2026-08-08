# Constraints Vocabulary

Reference guide for constraint categories, definitions, and how they shape architecture decisions.

---

## Purpose

Constraints are the boundaries within which architectures must fit. By naming constraints explicitly, teams ground decisions and catch when new constraints invalidate earlier choices.

---

## Constraint Categories

### 1. Organizational Constraints

**Team Size & Composition**
- Definition: How many people? What skills? How are they organized?
- Why it matters: Team size affects architectural complexity (Conway's Law: architecture mirrors team structure)
- Examples:
  - "3 backend engineers (1 senior, 2 junior), 1 frontend, 1 QA" → Monolith likely better than microservices
  - "8 independent teams" → Microservices necessary for independent deployments
  - "0 DevOps specialists" → Favor managed services (RDS, App Runner); avoid self-managed Kubernetes

**Team Experience & Maturity**
- Definition: What technologies is the team proficient in? What operational practices do they have?
- Why it matters: Learning curve consumes timeline; immature practices cause production incidents
- Examples:
  - "Team has no Kubernetes experience" → Cloud-native requires learning curve (3–6 months); fallback to managed services
  - "No on-call rotation established" → Avoid architectures requiring 24/7 monitoring + instant response
  - "Strong database optimization skills" → Can optimize single-large-database design; don't need sharding expertise

**Hiring & Timeline**
- Definition: Can you hire new people? By when?
- Why it matters: Architecture depends on team structure; if team can't scale, architecture can't scale
- Examples:
  - "6-month launch; can't hire until month 2" → Start with small team; architecture must scale with 5 → 10 people
  - "Senior hiring takes 3 months" → Architecture can't depend on senior-level skills until month 3+
  - "Hiring is open-ended" → Can evolve architecture based on people hired

---

### 2. Timeline & Business Constraints

**Time-to-Market (TTM)**
- Definition: When must the system be live (or MVP)?
- Why it matters: Tight TTM favors simpler architectures and known technologies
- Examples:
  - "6-month launch" → Don't learn new tech; use what team knows
  - "6-month MVP, then scale later" → Monolith acceptable; rewrite to microservices later if needed
  - "3-month MVP, then 12-month scale phase" → Plan for rewrite; architecture v1 ≠ production v1

**Feature Velocity**
- Definition: How fast must the team ship features?
- Why it matters: Tight coupling slows deployment; loose coupling enables fast iteration
- Examples:
  - "2–3 features per sprint" → Need independent deployment (feature flags, A/B testing)
  - "1 feature per quarter" → Can tolerate coordinated deployments (all services deploy together)
  - "Experiments must run in days" → Architecture must enable fast experiment branching + rollback

**Runway (Budget Time)**
- Definition: How long can the business sustain development before revenue/funding?
- Why it matters: Runway constrain scope, scale, and complexity
- Examples:
  - "18-month runway; Series B planned" → Can invest in foundational architecture; scale expected
  - "3-month runway; seeking seed funding" → Must ship MVP fast; architectural debt acceptable
  - "Profitable; indefinite runway" → Can invest in long-term scalability; technical excellence emphasized

---

### 3. Financial Constraints

**Infrastructure Budget (Annual)**
- Definition: How much can you spend on hosting, databases, third-party APIs, tools?
- Why it matters: Budget drives technology choices (serverless vs. self-managed; cloud vs. on-premises)
- Examples:
  - "$50K/year" → Managed services essential (RDS, App Runner, serverless); self-managed Kubernetes too expensive
  - "$500K/year" → Can run self-managed infrastructure; Kubernetes viable
  - "$10M/year" → Can afford redundancy, multi-region, specialized tools

**Operational Budget (Annual - People)**
- Definition: Staffing budget for operations, on-call, incident response?
- Why it matters: Complex architectures need dedicated ops; simple architectures can be run by engineers
- Examples:
  - "No ops budget" → Favor managed services; minimize operational burden on engineers
  - "1 FTE SRE" → Can handle 5–10 services; not 50+ microservices
  - "5 FTE SREs" → Complex architectures manageable; multi-region failover, self-healing possible

**Learning Budget (Ramp Time)**
- Definition: Can team spend time learning new technologies? Weeks? Months?
- Why it matters: New tech requires investment before productivity
- Examples:
  - "No learning budget; launch in 6 months" → Use known technologies only
  - "1 month learning sprint before coding" → Can learn 1 new major technology (not 3–4 new ones)
  - "6-month ramp-up acceptable" → Can invest in architectural maturity; new tech learning possible

---

### 4. Performance Constraints

**Latency (Response Time)**
- Definition: What's acceptable p99 latency for user-facing requests?
- Why it matters: Latency drives caching, database choice, geo-distribution
- Examples:
  - "<100ms p99" → Requires: caching, colocation, optimized DB; can't use async-all-the-way
  - "<500ms p99" → Moderate caching + DB optimization sufficient
  - "<5s p99" → Async acceptable; batch processing possible
  - "No target" → Constraint is not a bottleneck

**Throughput (Requests per Second)**
- Definition: How many concurrent requests or req/sec must the system handle?
- Why it matters: Throughput drives scaling strategy (vertical, horizontal, caching)
- Examples:
  - "100 concurrent users" → Single small server sufficient
  - "10K concurrent users" → Horizontal scaling + load balancer required
  - "100K concurrent users" → Distributed architecture required (multiple regions, CDN, sharding)

**Data Volume**
- Definition: How much data? How fast does it grow?
- Why it matters: Data volume drives database choice, retention policy, archiving strategy
- Examples:
  - "10GB total" → Single PostgreSQL instance sufficient
  - "1TB total, growing 100GB/month" → Archiving + indexing strategy needed
  - "1PB total" → Data warehousing architecture (OLAP vs. OLTP separation)

---

### 5. Reliability & Operational Constraints

**Availability SLA (Uptime %)**
- Definition: What percentage uptime is acceptable? (e.g., 99.5%, 99.99%)
- Why it matters: SLA drives redundancy, failover, monitoring requirements
- Examples:
  - "99% (7 days/year down time acceptable)" → Single point of failure acceptable; basic monitoring
  - "99.5% (3.6 days/year)" → Redundancy required; automated failover expected
  - "99.9% (8.76 hours/year)" → Multi-AZ, circuit breakers, rapid incident response
  - "99.99% (52 minutes/year)" → Multi-region, real-time replication, automated everything

**Recovery Time Objective (RTO)**
- Definition: After failure, how long to recover?
- Why it matters: RTO drives backup strategy, failover automation, runbook criticality
- Examples:
  - "RTO 24 hours" → Daily backups sufficient; manual restore acceptable
  - "RTO 1 hour" → Automated failover required; backup replication must be fast
  - "RTO 5 minutes" → Hot standby required; real-time replication critical
  - "RTO = near-zero" → Active-active required; no single failover point acceptable

**Recovery Point Objective (RPO)**
- Definition: Maximum acceptable data loss?
- Why it matters: RPO drives backup frequency, transaction durability, replication lag tolerance
- Examples:
  - "RPO 1 day" → Daily snapshots acceptable
  - "RPO 1 hour" → Hourly backups or continuous replication needed
  - "RPO = 0" → No data loss; synchronous replication required

---

### 6. Compliance & Regulatory Constraints

**Data Residency**
- Definition: Where can data be physically stored? (geographical regions)
- Why it matters: Data residency affects infrastructure choice (AWS region, on-premises, etc.)
- Examples:
  - "EU only (GDPR)" → All data must stay in EU regions; cross-border transfer impossible
  - "US only (CCPA)" → Data must stay in US; no EU replication
  - "Any region" → Full flexibility for geo-distribution + disaster recovery

**Data Classification**
- Definition: What types of data do you handle? (PII, PHI, payment, public, etc.)
- Why it matters: Data classification drives encryption, access control, audit requirements
- Examples:
  - "PII (names, emails, addresses)" → Encryption + access audit trail required
  - "Payment data" → PCI-DSS compliance mandatory; specific encryption + network isolation
  - "Health data (PHI)" → HIPAA compliance; 6-year audit trail; access logging

**Compliance Frameworks**
- Definition: Which standards apply? (SOC2, ISO 27001, HIPAA, PCI, GDPR, etc.)
- Why it matters: Compliance determines controls, monitoring, audit procedures
- Examples:
  - "SOC2 Type II" → Requires: user access controls, audit logging, change management, disaster recovery testing
  - "HIPAA" → Requires: encryption, access logs, 6-year retention, breach notification
  - "PCI-DSS" → Requires: network segmentation, encrypted payment processing, quarterly security assessments

**Data Retention & Deletion**
- Definition: How long must data be kept? When can it be deleted?
- Why it matters: Retention drives storage costs, compliance timelines, data archiving strategy
- Examples:
  - "7 years (GDPR/financial)" → Archive after 1 year; keep hot for 1 year, cold for 6 years
  - "Right to deletion within 30 days (GDPR)" → Delete pipelines must be automated + tested
  - "Forever (no deletion policy)" → Archive in data warehouse; remove from OLTP after retention window

---

### 7. Environmental Constraints

**Deployment Environment**
- Definition: Where will the system run? (cloud provider, on-premises, hybrid, edge)
- Why it matters: Deployment environment limits technology choices
- Examples:
  - "AWS" → Use AWS services; optimization for AWS cost model
  - "On-premises only" → No cloud; self-managed infrastructure; Kubernetes complexity acceptable
  - "Multi-cloud" → Avoid vendor lock-in; standardize on portable stacks (Kubernetes, Docker)
  - "Edge devices" → Limited CPU/RAM; lightweight architecture required

**Network Connectivity**
- Definition: What network conditions? (high-latency, high-jitter, offline-capable)
- Why it matters: Network affects synchronous call strategy, caching, sync patterns
- Examples:
  - "Low-latency office network (< 5ms)" → Synchronous RPCs acceptable
  - "High-latency internet (50–200ms)" → Async preferred; batch processing natural
  - "Intermittent connectivity (offline then sync)" → Event-driven + eventual consistency required

---

### 8. Security Constraints

**Authentication & Authorization**
- Definition: How are users/services authenticated? What authorization model?
- Why it matters: Auth affects component boundaries, secrets management, audit trails
- Examples:
  - "OAuth 2.0 for users; mTLS for services" → Requires: identity provider, certificate management
  - "API keys" → Simple but risky; requires rotation, audit logging
  - "Passwordless (OIDC)" → Requires: identity provider (Okta, Azure AD); enables multi-factor auth

**Data Encryption**
- Definition: Is encryption in transit and at rest required? What key management?
- Why it matters: Encryption affects performance, key rotation burden, disaster recovery
- Examples:
  - "Encryption in transit (TLS) + at rest (AES-256)" → Standard for regulated data
  - "Optional encryption" → Cost/performance optimization acceptable
  - "End-to-end encryption" → Client encrypts; server never has plaintext; limits search/indexing

**Network Isolation**
- Definition: Can services/data be segmented? Firewall rules?
- Why it matters: Network isolation limits blast radius of compromise
- Examples:
  - "Payment data isolated (separate network segment)" → PCI-DSS requirement
  - "No network segmentation" → All services can reach all databases; blast radius large
  - "VPC + security groups" → AWS standard; limits access but not air-gapped

---

### 9. Scale Constraints

**Initial Scale**
- Definition: How many users/requests/data at launch?
- Why it matters: Initial scale drives technology choices; might be overengineering if too low
- Examples:
  - "100 concurrent users, 1M records" → Single instance sufficient; optimize for development speed
  - "10K concurrent, 100M records" → Distributed architecture; scaling from day 1

**Projected Scale (12 Months)**
- Definition: Expected scale after 1 year?
- Why it matters: Must support growth without major rewrite
- Examples:
  - "100 → 10K concurrent users" → 100x growth; must plan for horizontal scaling
  - "10K → 100K concurrent" → 10x growth; linear scaling expected

**Expansion Scenarios**
- Definition: What are upside scenarios? (10x, 100x growth possibility)
- Why it matters: Architecture must not paint you into a corner
- Examples:
  - "Viral potential: 100x overnight possible" → Avoid single points of failure; autoscale essential
  - "Controlled growth: +50% annually" → Linear scaling acceptable; can refactor

---

## Constraint Interaction Map

**Constraints that often conflict:**

| Constraint 1 | Constraint 2 | Conflict | Resolution |
|-------------|-------------|----------|------------|
| Low latency (<100ms) | High availability (99.99%) | Both require distributed systems; adds complexity | Budget for ops; simplify where possible |
| Low infrastructure cost | High availability (99.99%) | HA costs ~2x; conflicts with bootstrap budget | Phase: start simple, invest in HA at scale |
| High feature velocity | Strong consistency | Microservices enable velocity; ACID transactions limit it | Eventual consistency for non-critical data |
| 6-month TTM | New technology | New tech adds learning curve; TTM is tight | Stick with known tech; plan rewrites |
| 3-person team | Multi-region disaster recovery | DR requires ops expertise; small team can't handle it | Defer DR to year 2; start single-region |
| $50K annual budget | Kubernetes (self-managed) | Self-managed K8s is expensive to operate | Use managed services instead |

---

## Constraint Priority Scoring

**When constraints conflict, which wins?**

Create a priority matrix for your project:

| Constraint | Priority | Rationale |
|-----------|----------|-----------|
| Time-to-market (6 months) | Critical | Launch date is non-negotiable |
| Team size (3 people) | Critical | Can't hire fast enough; constrains architecture |
| Cost ($50K/year) | High | Seed funding limited; cost affects scalability |
| Latency SLA (<500ms) | High | User experience; business requirement |
| Availability (99.5%) | Medium | Acceptable downtime ~2 hours/month |
| Compliance (SOC2) | High | Customer requirement for sales |

**When Cost vs. SLA conflicts:** SLA wins if it's customer-facing; otherwise optimize cost.

---

## Using Constraints as Decision Filter

**For every architecture decision:**

1. List all relevant constraints
2. For each constraint, check: Does this architecture **fit**, **violate**, or **require validation**?
3. If any violation, either: (a) change architecture, (b) challenge constraint (ask stakeholder), or (c) mark [REQUIRES VALIDATION]

**Example:**

| Constraint | Check | Status |
|-----------|-------|--------|
| 6-month TTM | Kubernetes learning curve 3 months; leaves only 3 months coding | ⚠️ TIGHT; plan training in month 1 |
| Team size 3 people | Kubernetes ops burden; need 1 FTE ops minimum | ❌ VIOLATES; recommend managed services |
| $50K annual budget | Self-managed K8s: $100+ K/year; exceeds budget | ❌ VIOLATES; recommend App Runner |
| Latency <500ms | K8s can support this; no violation | ✓ FITS |
| Availability 99.5% | K8s can support; RDS Multi-AZ supports | ✓ FITS |

**Decision:** Don't use Kubernetes; use AWS App Runner (managed) + RDS. Fits all constraints.

---

## Constraint Evolution

**Constraints change over time. Track changes:**

- **Baseline constraints (Day 1):** Document in ADR 0 or constraints doc
- **Monthly reviews:** Check if any constraints changed; if so, revalidate architecture
- **Trigger revalidation:** If funding changes, timeline changes, or team scales, re-run constraint check

Example:

```
Month 1: Team = 3 people → Architecture = Monolith + App Runner
Month 3: Series A closed; team scaled to 8 → Constraints changed → Revalidate
  - New constraint: "5 independent teams" → Recommend microservices
  - Old constraint: "$50K budget" → Now $500K budget → Kubernetes viable
Month 5: New deployment requirement "multi-region" → Adds complexity; revalidate HA strategy
```

---

## Constraints Documentation Template

```markdown
## Project Constraints

### Organizational
- Team size: 3 backend, 1 frontend, 1 QA
- Team experience: Node.js, React, PostgreSQL; no Kubernetes
- Timeline to hire: Can add 2 people by month 2

### Timeline & Business
- TTM (MVP): 6 months
- Feature velocity target: 2–3 features/sprint
- Runway: 18 months; Series B planned month 6

### Financial
- Infrastructure budget: $50K/year (must scale within budget if possible)
- Ops budget: 0 FTE SRE (engineers do ops)

### Performance
- Latency: <500ms p99 for search; <100ms p99 for product detail
- Throughput: 1K → 10K concurrent users over 12 months
- Data: 1M products; 100K customers

### Reliability
- Availability SLA: 99.5% (acceptable downtime ~2 hours/month)
- RTO: <1 hour (automated failover)
- RPO: <1 hour (hourly backups)

### Compliance
- Data residency: EU only (GDPR)
- Data types: Customer PII, payment tokens (no full card storage)
- Compliance: SOC2 Type II (required for enterprise sales)

### Environment
- Deployment: AWS (preferred; avoid lock-in if possible)
- Multi-region: Not required at launch; plan for year 2

### Scale
- Initial: 1K concurrent users; 1M products
- Year 1: 10K concurrent; 10M products
- Expansion possibility: Viral growth (10x overnight during flash sale)
```

Use this template for your project; share with architecture reviewers.
