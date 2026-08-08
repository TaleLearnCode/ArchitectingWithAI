# Process Flow Instructions

Template and guidelines for documenting process flows, decision points, error handling, and operational sequences.

---

## Purpose

A process flow captures **how** work gets done: the step-by-step sequence, decision points, error handling, timeouts, and actor responsibilities. Flows ensure consistency, enable testing, and guide operations.

---

## Key Principles

1. **Numbered steps** — Every action is numbered for reference.
2. **Clear actors** — Who takes each action: User, System, Service A, External API, etc.
3. **Decision branches** — Every IF/THEN explicitly states both paths.
4. **Error handling** — Every external call or risky step has a failure path.
5. **Timeouts** — External dependencies have explicit timeout and fallback.
6. **Testability** — Someone can trace through the flow and verify it works.

---

## Template Structure

### Header
- **Process Name:** Clear, concise name (e.g., "Customer Checkout Flow")
- **Scope:** What does this flow cover? What's out of scope?
- **Actors:** List all participants (User, Payment Service, Order Service, Payment Gateway, etc.)
- **SLA/Timing:** Any latency targets or timing expectations

### Happy Path (Normal Sequence)
Number each step:
```
Step N [ACTOR]: Do X
  INPUT: [What data/trigger starts this step]
  OUTPUT: [What data/signal this step produces]
```

Example:
```
Step 1 [CUSTOMER]: Submits checkout form with items and payment method
  INPUT: Cart contents (item IDs, quantities), payment method (card token)
  OUTPUT: HTTP POST to Payment Service
```

### Decision Points
For each decision, show both paths:
```
Step N [ACTOR]: Evaluate condition
  IF (condition is true):
    → Step X [SUCCESS PATH]
  ELSE:
    → Step Y [ERROR/ALTERNATE PATH] [HANDLING: specific action]
```

Example:
```
Step 3 [PAYMENT_SERVICE]: Validate card format
  IF (card passes Luhn check):
    → Step 4 [Call Payment Gateway]
  ELSE:
    → Step 10 [ERROR] [HANDLING: Return "Invalid card format" to customer; do NOT call gateway]
```

### Timeouts & Retries
For external dependencies:
```
Step N [SYSTEM]: Call External API
  TIMEOUT (X seconds): 
    IF no response within X seconds:
      → Step Y [HANDLING: Retry N times, then fallback]
  IF success:
    → Step X+1
  IF repeated failure:
    → Step Z [ERROR: Mark transaction for manual review]
```

Example:
```
Step 4 [PAYMENT_SERVICE]: Call Payment Gateway to authorize
  TIMEOUT (5 seconds):
    IF no response in 5 seconds:
      → Retry logic: Up to 2 attempts, then → Step 11 [HANDLING: Timeout; return "Payment processing delayed" to customer]
  IF success (approved):
    → Step 5 [Create Order]
  IF failure (declined):
    → Step 9 [ERROR: Return "Payment declined" to customer]
```

### Error Handling
Explicit error paths with recovery:
```
Step N [ERROR]: Condition that violates assumptions
  CAUSE: [What went wrong]
  IMPACT: [Who's affected; what's broken]
  HANDLING: [Specific action to recover or fallback]
  OWNER: [Who responds to this error in production]
  COMMUNICATION: [What message/alert is sent; to whom]
```

Example:
```
Step 9 [ERROR]: Payment Gateway is unreachable (connection timeout or 5xx error)
  CAUSE: Network outage, gateway maintenance, or service degradation
  IMPACT: Customers cannot complete checkout; orders queued; potential revenue impact
  HANDLING: 
    - Immediate: Return "Payment processing temporarily unavailable; please try again in 2 minutes" to customer
    - Alert on-call engineer: PagerDuty alert "Payment Gateway Down" with escalation
    - Check gateway status page; if maintenance, set customer expectation (ETA recovery)
    - If timeout > 5 minutes, execute fallback: queue order for manual authorization within 1 hour
  OWNER: On-call SRE or Payment team lead
  COMMUNICATION: Incident Slack channel; status page update if outage > 15 minutes
```

### Rollback or Compensation
If a step fails partway through, how do we undo?
```
Step N [COMPENSATION]: If step M fails, undo this
  UNDO ACTION: [Specific action to reverse the effect]
  VALIDATION: [How do we confirm undo succeeded?]
  OWNER: [Who can trigger this]
  TIMELINE: [How long does undo take; is it automatic or manual?]
```

Example:
```
Step N [COMPENSATION]: If Order Service fails after Payment Gateway approves
  UNDO ACTION: Call Payment Gateway /refund endpoint with authorization ID
  VALIDATION: Check refund status; confirm charge reversed in Payment Service
  OWNER: On-call engineer (can be triggered manually via runbook; ideally automated)
  TIMELINE: Automatic refund issued within 3 seconds; visible in customer account within 1 hour
```

### Monitoring & Observability
What signals tell us each step worked?
```
Step N [MONITORING]:
  - Log message: [Specific log line to expect]
  - Metric: [Counter, latency, etc. to track]
  - Alert condition: [When should we page?]
```

Example:
```
Step 4 [MONITORING]:
  - Log: "payment_gateway_call: method=POST path=/authorize status=200 latency_ms=250"
  - Metric: payment_gateway_latency_histogram (should be <500ms p99)
  - Alert: IF payment_gateway_error_rate > 5% for 1 minute → page on-call
```

---

## Complete Example: Checkout Flow

**Process Name:** Customer Checkout Flow  
**Scope:** From "Place Order" button to "Order Confirmation" page. Does NOT include shipping selection (separate flow).  
**Actors:** Customer (browser), API Gateway, Order Service, Payment Service, Payment Gateway, Email Service, Inventory Service  
**SLA:** Checkout must complete within 10 seconds p99; if Payment Gateway delays, notify customer by 5s mark.

---

### Steps

**Step 1 [CUSTOMER]:** Clicks "Place Order" button on checkout page  
  INPUT: Cart contents (items, quantities), delivery address, payment method  
  OUTPUT: HTTP POST /checkout with order payload  

**Step 2 [API_GATEWAY]:** Receives order request; validates authentication & rate limiting  
  IF valid authentication & not rate-limited:
    → Step 3  
  ELSE:
    → Step 15 [ERROR: Return 401 or 429 with message]  

**Step 3 [ORDER_SERVICE]:** Validate order constraints (items exist, quantities available)  
  IF validation passes:
    → Step 4  
  ELSE:
    → Step 14 [ERROR: Return "Item out of stock" or "Invalid item"]; do NOT reserve inventory  

**Step 4 [INVENTORY_SERVICE]:** Reserve inventory for each item  
  IF all items reserved successfully:
    → Step 5  
  ELSE:
    → Step 14 [ERROR: Return "Insufficient stock"; release any partial reservations]  

**Step 5 [PAYMENT_SERVICE]:** Authorize payment with Payment Gateway  
  TIMEOUT (5 seconds):
    IF no response in 5 seconds:
      → Retry up to 2 times with 2-second backoff  
      → IF still no response after retries: → Step 13 [ERROR: Return "Payment processing delayed"; queue for manual authorization]  
  IF authorization successful (approved):
    → Step 6  
  IF authorization fails (declined, insufficient funds, etc.):
    → Step 12 [ERROR: Return specific decline reason; release inventory reservation]  

**Step 6 [ORDER_SERVICE]:** Create order record in database  
  IF insert succeeds:
    → Step 7  
  ELSE:
    → Step 11 [ERROR: Database write failure; call Payment Service to void authorization; release inventory]  

**Step 7 [ORDER_SERVICE]:** Publish OrderCreated event to message queue  
  INPUT: Order ID, customer info, items  
  OUTPUT: Message published to fulfillment queue  
  TIMEOUT (2 seconds):
    IF publish fails: Log warning; proceed to step 8 (fulfillment team will poll eventually)  

**Step 8 [EMAIL_SERVICE]:** Send confirmation email to customer  
  INPUT: Order ID, customer email  
  TIMEOUT (3 seconds):
    IF email send fails: Log warning; customer sees confirmation in UI anyway  

**Step 9 [API_GATEWAY]:** Return success response to customer  
  OUTPUT: HTTP 200 with Order ID, confirmation details  
  CUSTOMER SEES: "Order Confirmation" page with order details  

---

### Error Paths

**Step 11 [ERROR: Database Failure]**  
  CAUSE: PostgreSQL write failed (full disk, connection lost, deadlock, etc.)  
  IMPACT: Order not created; payment reserved but order missing  
  HANDLING:
    - Immediate: Return "Order processing failed; please try again" to customer (do NOT retry within response window)
    - Call Payment Service /void endpoint to cancel authorization (within 30 seconds of error)
    - Manual intervention: SRE checks database logs; if deadlock, restarts connections; if disk full, archive old logs
    - If void fails: Flag order for manual refund review (attach Payment Service ref ID)
  OWNER: On-call SRE + Payment team  
  COMMUNICATION: Alert on-call; page if database unreachable for > 1 minute  

**Step 12 [ERROR: Payment Declined]**  
  CAUSE: Card declined (insufficient funds, fraud block, card expired, etc.)  
  IMPACT: Inventory reserved but not charged; order not created  
  HANDLING:
    - Immediate: Return specific decline reason to customer ("Card declined by issuer"; do NOT return generic message)
    - Release inventory reservation immediately (within 1 second)
    - Customer can retry with different payment method (no data loss)
  OWNER: Customer (self-service retry)  
  COMMUNICATION: Display decline reason in UI; no alert needed (expected behavior)  

**Step 13 [ERROR: Payment Gateway Timeout]**  
  CAUSE: Payment Gateway unreachable (network timeout, maintenance, service degradation)  
  IMPACT: Cannot authorize payment; order stuck in limbo  
  HANDLING:
    - Immediate: Return "Payment processing delayed; we'll complete your order within 1 hour" to customer
    - Queue order for manual authorization (capture card token, order ID, timestamp)
    - Alert on-call: "Payment Gateway unreachable; X orders queued for manual auth"
    - If gateway recovers: Process queued orders via async job (retry up to 2x before manual review)
    - If gateway down > 1 hour: Contact customer; offer alternate payment method or cancellation
  OWNER: On-call Payment engineer  
  COMMUNICATION: PagerDuty alert if queued orders > 10 or outage > 15 minutes  

---

### Compensation (Rollback)

**If Order Service fails after Payment Authorization (Step 6 fails):**
  ACTION: Call Payment Service /void to cancel authorization  
  VALIDATION: Confirm authorization canceled in Payment Gateway; check logs  
  TIMELINE: Automatic, completes within 2 seconds  
  FALLBACK: If void fails, flag order ID for manual refund review (attach authorization ref)  

**If Inventory Reservation not released (Step 4 succeeds but later step fails):**
  ACTION: Inventory Service /release endpoint with reservation ID  
  TIMELINE: Automatic if order creation fails; manual if stuck (SRE cleans up daily)  

---

### Monitoring

| Step | Log | Metric | Alert |
|------|-----|--------|-------|
| 5 | `payment_auth_call: status=OK latency_ms=XXX` | `payment_latency_p99` | IF latency > 2000ms |
| 6 | `order_created: order_id=XXX timestamp=XXX` | `orders_per_minute` | IF rate drops > 50% |
| 12 | `payment_declined: reason=insufficient_funds` | `decline_reason_distribution` | None (expected) |
| 13 | `payment_gateway_timeout: retry_attempt=2 queued=yes` | `gateway_timeouts_per_hour` | IF > 5 timeouts/hour |

---

## Quality Checklist

- [ ] Every step has a clear actor (not ambiguous "system")
- [ ] Every external dependency has a timeout (not "wait until it responds")
- [ ] Every error case has an explicit handling path (not "retry indefinitely")
- [ ] Error paths show who's responsible and how to communicate
- [ ] Compensation steps exist for critical failure scenarios (not just hope it doesn't happen)
- [ ] Monitoring signals defined for each critical step
- [ ] Flow is testable (someone can trace through and verify each step)
- [ ] No invented timeouts or SLAs (all grounded in real constraints)
- [ ] Timing is realistic (e.g., email < 3s is optimistic; flag if critical-path)

---

## Anti-Patterns

❌ **Vague actors:** "System processes order" (unclear who/what)  
✓ **Specific actors:** "Order Service: Validates order constraints"

❌ **Missing error handling:** Flow only shows happy path  
✓ **Complete error paths:** Each external call has timeout, retry, and fallback

❌ **Unrealistic timeouts:** "Payment Gateway responds within 100ms"  
✓ **Realistic timeouts:** "Payment Gateway timeout 5s; retry 2x; if still down, queue for manual auth"

❌ **Silent failures:** "If inventory check fails, skip to next step"  
✓ **Explicit failures:** "If inventory fails, roll back payment and return error to customer"

---

## Common Process Flows to Document

1. **Customer checkout** — Payment, order creation, confirmation
2. **Order fulfillment** — Picking, packing, shipping
3. **Incident response** — Detection, escalation, resolution, postmortem
4. **Data pipeline** — Ingestion, validation, enrichment, export
5. **Backup & disaster recovery** — Trigger conditions, steps, validation
6. **User onboarding** — Account creation, verification, initial setup
