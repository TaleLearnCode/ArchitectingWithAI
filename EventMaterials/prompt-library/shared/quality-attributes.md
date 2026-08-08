# Quality Attributes Reference

ISO 25010 quality attributes and how they apply to architecture decisions.

---

## Purpose

Quality attributes (scalability, security, performance, etc.) are non-functional requirements. This reference defines what each means, when it matters, and how to measure it.

---

## Quality Attributes (ISO 25010)

### 1. **Performance**

**Definition:** How fast does the system respond?

**Key Metrics:**
- Response latency (p50, p99 latency)
- Throughput (requests per second)
- Data processing speed

**When It Matters:**
- User-facing operations (must be <2 seconds for good UX)
- Real-time systems (trading, gaming, live updates)
- High-throughput systems (analytics, data pipelines)

**Architecture Impact:**
- Requires caching, indexing, async processing
- Affects database choice (in-memory vs. disk)
- Drives use of CDNs, edge computing

**How to Specify:**
- "API latency <500ms p99 for 100K concurrent users"
- Not: "Fast API"

**Example SLOs (Service Level Objectives):**
- Search results: <100ms p99
- Product pages: <200ms p99
- Checkout: <500ms p99

---

### 2. **Reliability**

**Definition:** Does the system work as expected without failures?

**Key Metrics:**
- Mean Time Between Failures (MTBF)
- Mean Time To Recovery (MTTR)
- Error rate (% failed requests)
- Availability (uptime %)

**When It Matters:**
- Business-critical systems (payment, healthcare, finance)
- Customer-facing systems (can't afford downtime)
- Production systems (failures cost money)

**Architecture Impact:**
- Requires redundancy, failover, monitoring
- Affects technology choices (managed services vs. self-managed)
- Drives incident response procedures

**How to Specify:**
- "99.5% availability (acceptable downtime ~2 hours/month)"
- "MTTR <5 minutes for database failure"
- Not: "Highly available"

**Example SLOs:**
- Payment system: 99.99% availability (SLA violation = revenue loss)
- Reporting system: 99% availability (lower priority)

---

### 3. **Scalability**

**Definition:** How does performance change as load increases?

**Key Metrics:**
- Vertical scaling limit (single server max throughput)
- Horizontal scaling factor (how many servers needed for 10x load)
- Cost scaling (does cost grow linearly with users?)
- Data scaling (how much data before redesign needed)

**When It Matters:**
- Growing user bases (startups expecting 10x growth)
- Bursty traffic (flash sales, viral events)
- Long-term roadmap (planning for year 3)

**Architecture Impact:**
- Affects database design (sharding? partitioning?)
- Determines distributed vs. monolithic (is centralization bottleneck?)
- Influences cloud vs. on-premises (elasticity)

**How to Specify:**
- "Scales horizontally from 1K to 100K concurrent users"
- "Cost scales linearly with users (no cliff at 10K or 100K)"
- "Database bottleneck appears at 1M records (mitigation: sharding)"
- Not: "Scalable to millions of users"

**Example SLOs:**
- "Handles 10–100x growth without rewrite"
- "Deployment takes <5 min regardless of scale"

---

### 4. **Security**

**Definition:** Is sensitive data protected? Are attacks prevented?

**Key Metrics:**
- Authentication (who can access?)
- Authorization (what can each user do?)
- Encryption (data at rest + in transit)
- Audit trail (who did what, when?)
- Breach response time

**When It Matters:**
- Handling customer/user data (PII, healthcare, financial)
- Compliance requirements (GDPR, PCI, HIPAA)
- Public-facing systems (attack surface)

**Architecture Impact:**
- Requires network segmentation, encryption keys
- Affects authentication/authorization design
- Impacts data flow (where sensitive data lives)

**How to Specify:**
- "PII encrypted in transit (TLS 1.3) and at rest (AES-256)"
- "GDPR compliance: data residency EU only, right to deletion in 30 days"
- "PCI-DSS: payment data isolated, never stored in logs"
- Not: "Secure"

**Example SLOs:**
- "Breach detection time: <1 hour"
- "Incident response SLA: <4 hours"

---

### 5. **Maintainability**

**Definition:** How easy is it to understand, fix, and modify the system?

**Key Metrics:**
- Time to deploy a bug fix
- Time to understand a module
- Code duplication %
- Cyclomatic complexity

**When It Matters:**
- Long-lived systems (>3 years)
- Large teams (>5 engineers, coordination matters)
- Complex domains (business logic intricate)

**Architecture Impact:**
- Clear component boundaries (modules don't depend on each other)
- Documentation quality
- Modular design vs. monolithic

**How to Specify:**
- "Bug fix deployed within 4 hours (P1 severity)"
- "New engineer productive on feature in <2 weeks"
- "50% code duplication removed by year 2"
- Not: "Easy to maintain"

**Example SLOs:**
- "Mean time to fix bug: <24 hours"
- "Documentation updated within 1 week of code change"

---

### 6. **Testability**

**Definition:** How easy is it to verify the system works correctly?

**Key Metrics:**
- Test coverage %
- Time to run test suite
- Time to write test for new feature
- Number of flaky tests

**When It Matters:**
- High-risk systems (failures are expensive)
- Rapid iteration (CI/CD deployment many times/day)
- Complex business logic (easy to introduce bugs)

**Architecture Impact:**
- Affects component isolation (independent testing)
- Determines mocking/stubbing needs
- Impacts testing infrastructure (test environments)

**How to Specify:**
- "Unit test coverage >80%"
- "Integration tests pass in <5 minutes"
- "E2E tests for critical paths (checkout, payment)"
- Not: "Testable"

**Example SLOs:**
- "All code changes have tests before merge"
- "Test suite completion time: <10 minutes"

---

### 7. **Usability**

**Definition:** Is the system easy for users to use?

**Key Metrics:**
- User task completion rate
- Time to learn (onboarding)
- Error recovery time
- User satisfaction score

**When It Matters:**
- User-facing systems (customer retention depends on UX)
- Complex interfaces (medical, financial)
- Accessibility requirements (WCAG compliance)

**Architecture Impact:**
- Affects API design (ease of use for developers if B2B)
- Impacts UX consistency (consistent across devices)
- Determines responsiveness (UX lag frustrates users)

**How to Specify:**
- "Checkout flow: 3 clicks to order (target: 5 clicks)"
- "WCAG 2.1 Level AA compliance (accessible to disabled users)"
- "Onboarding: 90% of new users complete first action in <5 minutes"
- Not: "User-friendly"

---

### 8. **Availability**

**Definition:** Is the system accessible when needed?

**Key Metrics:**
- Uptime % (fraction of time system works)
- Planned downtime (maintenance windows)
- Unplanned downtime (outages)
- Geographic availability (available in all regions?)

**When It Matters:**
- Business-critical systems (payment, order processing)
- 24/7 operations (no maintenance windows acceptable)
- Global systems (users in multiple time zones)

**Architecture Impact:**
- Requires redundancy, monitoring, incident response
- Affects multi-region strategy
- Determines maintenance procedures (zero-downtime deployments)

**How to Specify:**
- "99.5% availability (3.6 days/year downtime acceptable)"
- "Planned downtime: Saturday 2–3am UTC only (outside business hours)"
- "Available in: AWS US, EU, APAC regions"

---

### 9. **Interoperability**

**Definition:** Can this system work with other systems?

**Key Metrics:**
- Number of integrations supported
- Data format compatibility
- API versioning (backward compatible?)
- Vendor lock-in risk

**When It Matters:**
- Ecosystems of services (microservices talking to each other)
- B2B integrations (partners' systems call ours)
- Data migration (switching providers later)

**Architecture Impact:**
- Affects API design (RESTful? GraphQL? gRPC?)
- Determines data format (JSON? XML? proprietary?)
- Impacts vendor choice (cloud provider lock-in?)

**How to Specify:**
- "REST API with JSON responses, compatible with OpenAPI 3.0"
- "Supports Shopify, WooCommerce, custom integrations"
- "Database: PostgreSQL (portable, no vendor lock-in)"
- Not: "Compatible with everything"

---

### 10. **Compatibility**

**Definition:** Does the system work across different environments (OS, browsers, devices)?

**Key Metrics:**
- Supported browsers/versions
- Supported operating systems
- Mobile/desktop compatibility
- Backward compatibility (old API versions still work)

**When It Matters:**
- Web applications (cross-browser, cross-device)
- Mobile apps (iOS/Android support)
- Long-lived systems (upgrading user base gradually)

**Architecture Impact:**
- Affects UI/UX design (responsive, accessible)
- Impacts API versioning (old clients must still work)
- Determines platform support (Windows? Mac? Linux?)

**How to Specify:**
- "Supports: Chrome, Firefox, Safari, Edge (latest 2 versions)"
- "Mobile: iOS 13+, Android 8+"
- "API v1 supported until 2025; v2 is new standard"

---

### 11. **Efficiency / Resource Usage**

**Definition:** How much computational resources does the system use?

**Key Metrics:**
- CPU usage per request
- Memory footprint
- Bandwidth/data transfer
- Power consumption
- Storage footprint

**When It Matters:**
- Cost-sensitive (limited infrastructure budget)
- Edge computing / mobile (limited CPU/memory)
- Energy-conscious (data center power costs)
- Cost-optimized (AWS/cloud billing)

**Architecture Impact:**
- Affects technology choices (lightweight languages/frameworks?)
- Determines infrastructure sizing (small instances vs. large)
- Impacts design (in-memory caching vs. disk caches)

**How to Specify:**
- "Average memory per instance: <500MB"
- "CPU usage <50% at peak load (headroom for spikes)"
- "Data transfer: <1 GB/day (optimize for bandwidth cost)"
- Not: "Efficient"

---

### 12. **Modifiability / Flexibility**

**Definition:** How easy is it to change the system for new requirements?

**Key Metrics:**
- Time to add a new feature
- Lines of code affected by requirement change
- % of components needing change (coupling measure)
- Feature branch complexity (merge conflicts?)

**When It Matters:**
- Rapidly evolving products (frequent feature changes)
- Long-lived systems (requirements will change)
- Multiple customer variants (customization needed)

**Architecture Impact:**
- Requires modular design, decoupling
- Impacts technology choices (frameworks with extension points)
- Determines feature flag strategy (deploy but don't activate)

**How to Specify:**
- "Time to add new payment method: <3 days (estimate)"
- "Feature can be added to single service without changing others"
- "Feature flags enable fast rollback if issues arise"

---

## Quality Attribute Priority Matrix

**For your project, prioritize:**

| Attribute | Priority | Constraint It's Driven By |
|-----------|----------|--------------------------|
| Performance | HIGH | SLA: <500ms p99 latency requirement |
| Reliability | HIGH | Business impact: checkout failures lose revenue |
| Security | HIGH | Compliance: GDPR + PCI-DSS required |
| Scalability | MEDIUM | Growth projection: 1K → 100K over 12 months |
| Maintainability | MEDIUM | Team size: 8 engineers need code clarity |
| Testability | MEDIUM | Release frequency: 2x per week requires confidence |
| Usability | LOW | Internal tool; UX secondary |
| Availability | HIGH | SLA: 99.5% uptime target |

**Rule:** Focus on HIGH priority attributes in architecture. Address MEDIUM in implementation. LOW can be deferred.

---

## Trade-offs Between Attributes

**Common conflicts:**

| Attribute 1 | Attribute 2 | Trade-off | Resolution |
|-------------|------------|----------|-----------|
| Performance | Security | Encryption adds latency | Accept latency hit; use TLS 1.3 |
| Scalability | Maintainability | Distributed systems add complexity | Modular design + clear interfaces |
| Reliability | Cost | Redundancy expensive | Start simple; add redundancy at scale |
| Performance | Maintainability | Optimization makes code harder to understand | Document critical paths; optimize later if needed |
| Availability | Testability | Distributed systems hard to test | Invest in testing infrastructure; chaos engineering |

**Strategy:** Identify which attribute matters most; accept tradeoff in lower-priority attribute.

---

## Quality Attribute Measurement Template

**For each HIGH-priority attribute:**

```markdown
### [Attribute Name]

REQUIREMENT: [SLA or target]
MEASUREMENT: [How will we know if we're meeting it?]
MONITORING: [What metrics, dashboards, alerts?]
BASELINE: [Current state before improvement]
TARGET: [Where we want to be]
SUCCESS CRITERIA: [Specific, testable threshold]
```

**Example:**

```markdown
### Performance

REQUIREMENT: API latency <500ms p99 for 100K concurrent users
MEASUREMENT: Capture response time for every request; calculate p99 percentile
MONITORING: Prometheus histogram; Grafana dashboard; alert if p99 > 800ms
BASELINE: Currently not measured; need load test
TARGET: <500ms p99
SUCCESS CRITERIA: Load test simulating 100K concurrent users; p99 <500ms for 24-hour run
```

---

## Common Quality Attribute Mistakes

❌ **Vague:** "System should be scalable"  
✓ **Specific:** "Horizontal scaling from 1K to 100K concurrent users; database write bottleneck at 1M concurrent (plan sharding for year 2)"

❌ **Conflicting without tradeoff:** "Highly secure AND very fast"  
✓ **With tradeoff:** "TLS encryption required (security priority); accept latency hit of <50ms (performance slightly reduced but acceptable)"

❌ **Unmeasurable:** "Reliable system"  
✓ **Measurable:** "99.5% availability; <5 minute MTTR for database failure"

---

## Quality Attributes Mapping to Sections

| Prompt Library Section | Quality Attributes to Consider |
|----------------------|--------------------------------|
| 01 Requirements | Define quality attributes as non-functional requirements |
| 02 Architecture Options | Compare options on quality attributes |
| 03 Adversarial Review | Challenge assumptions about quality metrics |
| 04 Documentation | Document how architecture achieves quality targets |
| 05 Squad | Each agent validates quality for their domain |
| 06 Governance | Anti-slop: catch vague quality claims |

---

## Reference: ISO 25010 Full Standard

For complete details, consult ISO/IEC 25010:2015 Standard. Key takeaway: Define quality attributes explicitly; don't assume "good quality" is understood.
