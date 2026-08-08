# Risk Document Instructions

Template for documenting risks: probability, impact, mitigation, contingency, monitoring, and ownership.

---

## Purpose

A Risk Register captures potential problems: what could go wrong, how likely it is, what the impact would be, how we prevent it, and how we respond if it occurs.

---

## Template Structure

### Header
- **Risk ID:** Unique identifier (e.g., RISK-001)
- **Title:** One sentence, specific (not "Performance risk"; say "Database write throughput bottleneck at >10K orders/sec")
- **Category:** Infrastructure, Operational, Compliance, Dependency, Team, Security
- **Owner:** Who drives mitigation? (name or role)
- **Last Updated:** Date of last review

### 1. Risk Description

**What could go wrong?** Be specific. Reference architecture constraints.

**Example:**
```
Payment Gateway (Stripe) experiences outage lasting >15 minutes. 
Customers cannot complete checkout. 
Our system queues orders in memory; if outage >30 min, queued orders are lost on server restart.
```

❌ **Weak:** "Payment processing fails"  
✓ **Strong:** "Payment Gateway outage >30 min causes loss of queued orders if app server restarts before manual processing"

### 2. Trigger

**What event would cause this risk to manifest?**

```
Payment Gateway HTTP 5xx errors OR
Connection timeouts (TCP timeout) OR
Payment Gateway status page reports "Incident" OR
Our error logs show payment_auth_timeout_count > 100/minute
```

### 3. Probability Assessment

**Current Probability:** Low / Medium / High  
**Justification:** Ground in real data or architecture facts.

**Example:**
```
Current Probability: MEDIUM

Justification:
- Stripe SLA is 99.99% (4.3 hours downtime/year)
- We've had 2 outages in the past 18 months (both <5 min)
- We have no fallback; every outage blocks all orders
- Our queuing strategy (in-memory) is fragile: restart loses data
- Probability: 1–2 outages/year * 30 min avg duration = ~1 hour/year impact
```

### 4. Impact Assessment

**What happens if the risk occurs?**

| Dimension | Impact | Severity |
|-----------|--------|----------|
| **Revenue** | Customers cannot complete checkout; estimated $5K per hour lost (100 orders/hr * $50 avg order) | High |
| **Technical** | Orders queued in memory; if app crashes, data lost; manual recovery required | High |
| **Operational** | On-call must process queued orders manually (1 order/min * X hours); escalates to CEO if >2 hours | High |
| **Reputation** | If outage >1 hour, customers complain on social media; trust erosion | Medium |
| **Compliance** | GDPR data retention: if we lose order records, audit trail broken | Medium |

**Total Impact Severity:** HIGH (financial + operational impact)

### 5. Current Mitigation (What We're Already Doing)

```
✓ Use Stripe as payment processor (they have 99.99% SLA; better than we could build)
✓ Implement timeout on Payment Gateway calls (5 seconds; retry 2x before failing)
✓ Monitor Payment Gateway error rate; alert on-call if > 5% errors for 1 minute
✓ Have incident runbook for payment outage (page on-call; communicate to customers)

✗ NOT MITIGATING:
- No fallback payment method (can't use secondary processor without integration)
- Orders queued in memory are not persisted (app restart loses data)
- No circuit breaker (after 3 failures, we could fail-open and queue for manual auth)
```

**Current Risk Level After Existing Mitigation:** MEDIUM-HIGH  
(We have alerting and timeout, but no fallback for extended outages)

### 6. Recommended Additional Mitigation

**Primary Mitigation:** Persist order queue to database

```
Implementation:
1. Add order status "payment_pending" in PostgreSQL
2. When Payment Gateway times out, insert order into payment_pending table (atomic with order creation)
3. Async job polls payment_pending table; retries Stripe auth every 2 minutes for 1 hour
4. If retry succeeds, update order status to "payment_confirmed"
5. If 1 hour expires without success, flag order for manual review (send to support team)

Cost: ~2 hours engineering (database schema, async retry job, monitoring)
Timeline: 1 sprint
Effort: Low
Effectiveness: Moves risk from "revenue loss" to "manual support burden"
```

**Secondary Mitigation:** Implement circuit breaker

```
Implementation:
1. After 5 consecutive Payment Gateway failures in 1 minute, activate circuit breaker
2. Circuit breaker: New requests immediately fail-open (queue to payment_pending table)
3. Circuit breaker resets after 5 minutes of successful calls
4. Alert: "Payment Gateway circuit breaker activated; X orders queued for manual auth"

Cost: ~1 hour engineering
Timeline: 1 sprint
Effort: Very low
Effectiveness: Prevents cascading failures; queues orders instead of failing customers
```

**Tertiary Mitigation (Optional):** Secondary payment processor

```
Implementation:
1. Integrate Adyen or Paypal as secondary processor
2. If Stripe fails, automatically try Adyen
3. Requires reconciliation (orders might succeed in both systems)

Cost: ~40 hours engineering (integration, reconciliation, testing)
Timeline: 2–3 sprints
Effort: High
Effectiveness: Eliminates single point of failure; but adds operational complexity (two processors to monitor/reconcile)
ROI: Low (primary + secondary mitigation above probably sufficient)
```

### 7. Contingency Plan (If Risk Occurs Anyway)

**If Payment Gateway outage is detected:**

| Time | Action | Owner | Expected Outcome |
|------|--------|-------|------------------|
| T+0 | PagerDuty alert fires; on-call opens incident channel | On-call | Team assembled; clock starts |
| T+1 | Check Stripe status page + our error logs to confirm outage | On-call | Diagnosis: "Stripe down" vs. "our bug" |
| T+3 | If Stripe issue, post in internal Slack: "Payment processing delayed; ETA from Stripe status page" | On-call | Team aware; customer support prepared |
| T+5 | Customer support begins fielding complaints; reply: "Payment processing temporarily unavailable; we're working with Stripe; ETA [time]" | Support Lead | Customer expectations set |
| T+15 | If outage persists, enable manual auth queue: Support team processes orders via Stripe dashboard manually (1 order/min) | Payment Team | Revenue recovery begins |
| T+30 | If still down, CEO notified; prepare public status page update | On-call Lead + CEO | Transparency; trust maintenance |
| T+60+ | Post-incident: Analyze root cause, implement recommended mitigation above | Engineering Lead | Prevent recurrence |

**Recovery procedure:**
1. Once Stripe recovers, async retry job automatically processes queued orders
2. Verify all queued orders eventually succeeded or were marked for manual review
3. Reconcile order records with Stripe; correct any duplicates
4. Generate report: "X orders delayed; Y manual audits needed; Z loss of revenue"

### 8. Monitoring Strategy

**How do we detect this risk is occurring?**

| Signal | Threshold | Alert Severity | Action |
|--------|-----------|-----------------|--------|
| Payment Gateway error rate | >5% for 1 min | Page on-call | Immediate investigation |
| Payment Gateway latency p99 | >5 sec for 1 min | Page on-call | Investigate Stripe; check circuit breaker |
| Orders in payment_pending table | >50 (if we implement mitigation 1) | Warning | Expected during outage; not actionable unless persists |
| Stripe status page status | ≠ "Operational" | Informational | Confirm external outage; adjust alerts |
| Circuit breaker active | Any | Page on-call | Circuit breaker triggered; Stripe likely unavailable |

**Dashboards to watch:**
- Stripe error rate (query from our logs, not Stripe API)
- Payment latency distribution (p50, p99)
- Order status distribution (% payment_pending, % payment_confirmed, % failed)

### 9. Review Cadence

- **Trigger-based review:** If threshold breached or incident occurs, review immediately
- **Regular review:** Monthly (check if new mitigations implemented; any new triggers)
- **Annual review:** Full risk reassessment (update probability, impact, effectiveness)

### 10. Sign-Off

- **Identified by:** [Architect name]
- **Reviewed by:** [Risk owner, Lead engineer]
- **Approved by:** [CTO or authority]
- **Review date:** [YYYY-MM-DD]
- **Next review:** [YYYY-MM-DD + 30 days]

---

## Quality Checklist

- [ ] Risk description is specific (tied to architecture, not generic)
- [ ] Probability is grounded in real data or architecture facts (not invented)
- [ ] Impact assessment covers multiple dimensions (financial, technical, operational, reputation)
- [ ] Current mitigation is explicit (what we're already doing)
- [ ] Recommended mitigation is actionable and costed (hours, timeline, effort)
- [ ] Contingency plan includes timeline, owners, and expected outcomes
- [ ] Monitoring strategy has specific thresholds (not "monitor for issues")
- [ ] Review cadence is explicit and realistic
- [ ] Owner is named; accountability is clear

---

## Anti-Patterns

❌ **Vague risk:** "Payment system might fail"  
✓ **Specific risk:** "Payment Gateway outage >30 min queues orders in memory; app restart loses queued orders; results in ~$5K revenue loss"

❌ **Ungrounded probability:** "High risk because payment is critical"  
✓ **Grounded probability:** "Medium risk: Stripe SLA 99.99% = 4.3 hours/year; we've had 2 outages in 18 months; no fallback means each outage blocks all orders"

❌ **Generic mitigation:** "Improve reliability" or "Add redundancy"  
✓ **Specific mitigation:** "Persist order queue to PostgreSQL; async retry job every 2 min for 1 hour; ~2 hours engineering cost"

❌ **Missing contingency:** "We'll handle it during incident"  
✓ **Documented contingency:** "T+1: Check status page; T+5: Support team replies to customers; T+15: Enable manual auth queue; T+60: Post-incident analysis"

---

## Common Risk Categories to Document

1. **Infrastructure dependency** — Payment gateway, cloud provider, database
2. **Operational** — On-call burnout, incident response gap, deployment failure
3. **Compliance** — Data residency, audit trails, GDPR/HIPAA violation
4. **Team** — Key person leaves, skill gap, learning curve miss
5. **Security** — Data breach, credential exposure, DDoS
6. **Performance** — Database bottleneck, API latency SLA miss, scaling failure
7. **Data** — Data corruption, backup failure, recovery failure
8. **Business** — Feature scope creep, timeline miss, budget overrun
