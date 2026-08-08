# Squad Roles

Role definitions for multi-agent architecture teams. Use these to assign responsibilities and clarify handoffs between agents.

---

## Purpose

When a squad of AI agents or cross-functional team members collaborates on architecture, clarity about roles prevents duplicated work and ensures handoffs are clean.

---

## Core Roles

### Role 1: **Morpheus — Lead Architect**

**Responsibilities:**
- Define architectural principles and strategic fit
- Review technology choices for long-term viability
- Approve major architecture decisions (ADRs)
- Challenge assumptions; demand evidence
- Synthesize recommendations into final architecture package

**Inputs This Role Needs:**
- Clarified requirements (from Requirements Analyst)
- Multiple architecture options with tradeoffs (from Explorer)
- Risk analysis and challenges (from Adversarial Reviewer)
- Documented assumptions and validation plans (from Analyst)

**Outputs This Role Produces:**
- Approved ADR set
- Final architecture specification
- Decision rationale for stakeholders
- Constraints inventory for team

**Success Criteria:**
- Architecture is grounded in stated constraints (not generic)
- All major decisions documented with rationale
- Stakeholder concerns addressed
- Team can articulate "why we chose this"

**Escalation Triggers:**
- Conflicting constraints (asks stakeholder to prioritize)
- Technology gap (requires team learning > 3 months)
- Budget/timeline misalignment (escalates to executive sponsor)

---

### Role 2: **Neo — SRE / Operations Authority**

**Responsibilities:**
- Review architecture for operability
- Identify failure modes and recovery procedures
- Design monitoring, alerting, and runbooks
- Estimate operational burden (FTE required)
- Ensure incident response feasible

**Inputs This Role Needs:**
- Architecture specification (from Spec Generator)
- Risk register (from Risk Analyst)
- Performance targets (from Requirements Analyst)
- Deployment environment constraints (from Lead)

**Outputs This Role Produces:**
- Operational requirements document
- Runbook templates for critical failure paths
- Monitoring & alerting strategy
- SRE resource estimate
- Deployment & release procedures

**Success Criteria:**
- Every component has documented failure recovery
- Monitoring strategy is specific (not "monitor for issues")
- On-call burden is acceptable
- Team can run architecture without 24/7 expert presence

**Escalation Triggers:**
- Architecture has >3 single points of failure (asks for redesign)
- Monitoring/observability gaps exist (asks for instrumentation plan)
- Ops burden > 1 FTE (negotiates with Lead for simplification)

---

### Role 3: **Trinity — Cost & Resource Lead**

**Responsibilities:**
- Estimate total cost of ownership (infrastructure + ops + learning)
- Validate cost assumptions
- Identify cost trade-offs
- Plan cost scaling as system grows
- Negotiate budget with stakeholders

**Inputs This Role Needs:**
- Technology choices and quantities (from Spec Generator)
- Operational requirements (from SRE)
- Team learning plan (from Requirements Analyst)
- Scale projections (from Requirements Analyst)

**Outputs This Role Produces:**
- Cost breakdown (infrastructure + ops + learning)
- Year-1 budget estimate
- 3-year cost projection
- ROI analysis vs. alternatives
- Cost optimization opportunities

**Success Criteria:**
- Cost estimate is within 20% of actual (not wild guesses)
- Budget constraints explicitly documented
- Cost trade-offs clearly communicated
- Scaling costs modeled (linear? exponential? cliffs?)

**Escalation Triggers:**
- Cost exceeds budget by >20% (asks Lead to simplify)
- Hidden costs discovered mid-project (costs ops/learning underestimated)
- Scale costs become non-linear (requires architecture redesign)

---

### Role 4: **Tank — Research & Requirements Analyst**

**Responsibilities:**
- Extract and clarify requirements
- Surface ambiguities and contradictions
- Document assumptions and validation plans
- Fact-check claims; demand evidence
- Maintain assumptions register and decision log

**Inputs This Role Needs:**
- Stakeholder input (project description, constraints, goals)
- Generated architecture documents (for validation)
- Team feedback and questions

**Outputs This Role Produces:**
- Functional & non-functional requirements (clarified)
- Constraints inventory (with priority)
- Ambiguities & clarifying questions for stakeholders
- Assumptions register (with confidence levels)
- Validation plans for risky assumptions
- Decision log (tracks all major decisions + rationale)

**Success Criteria:**
- Requirements are specific (testable, not vague)
- All assumptions documented and flagged
- Ambiguities escalated to stakeholders (not buried)
- Evidence cited for every claim in architecture docs
- Decision rationale traceable

**Escalation Triggers:**
- Requirements are contradictory (asks stakeholder to choose)
- Critical assumptions unvalidated (flags [HIGH_PRIORITY_VALIDATION_NEEDED])
- Evidence missing for key decisions (asks for research or marks as speculative)

---

### Role 5: **Switch — Product & Velocity Advocate**

**Responsibilities:**
- Ensure architecture enables fast feature delivery
- Challenge architectural complexity that slows development
- Design feature flag / A/B testing strategy
- Plan deployment & rollback procedures
- Advocate for engineer productivity

**Inputs This Role Needs:**
- Architecture specification (from Spec Generator)
- Deployment procedures (from SRE)
- Technology choices (from Lead)
- Feature roadmap / velocity targets (from Product)

**Outputs This Role Produces:**
- Feature delivery strategy (independent deployments? coordinated?)
- Feature flag architecture
- A/B testing framework
- Rollback procedures
- Estimated engineering velocity (days per feature)

**Success Criteria:**
- Features can be deployed independently (no coordination wait)
- Feature flags enable safe rollouts
- Rollback procedure < 5 minutes
- Engineering velocity acceptable (target: X features/sprint)
- Technical debt management plan exists

**Escalation Triggers:**
- Architecture requires coordinated deployments (asks for loosening coupling)
- Feature flag strategy adds complexity (asks to simplify)
- Velocity target unachievable (negotiates with Lead for simplification)

---

### Role 6: **Agent Smith — Security & Compliance**

**Responsibilities:**
- Review architecture for security gaps
- Ensure compliance requirements are met
- Define data protection strategy
- Design access control & audit trails
- Threat model critical components

**Inputs This Role Needs:**
- Architecture specification (from Spec Generator)
- Data flow diagrams (from Spec Generator)
- Compliance requirements (from Requirements Analyst)
- External dependencies list (from Spec Generator)

**Outputs This Role Produces:**
- Security architecture (encryption, authentication, authorization)
- Data protection & governance strategy
- Compliance checklist (GDPR, PCI, HIPAA, SOC2, etc.)
- Threat model & mitigations
- Access control & audit procedures

**Success Criteria:**
- Every data flow encrypted (transit + at rest)
- Blast radius of compromise limited (defense in depth)
- Compliance requirements addressed (not deferred)
- Access audit trail exists
- Security assumptions validated

**Escalation Triggers:**
- Sensitive data flow unencrypted (asks for redesign)
- Single point of failure in security (asks for redundancy)
- Compliance violations (escalates to legal/exec)

---

## Handoff Checklist

**When one role hands off work to the next, verify completeness:**

### From Requirements Analyst to Architect/Explorer
- [ ] Requirements are specific (not vague)
- [ ] Constraints are prioritized
- [ ] Ambiguities escalated to stakeholders
- [ ] Assumptions register drafted
- [ ] No [REQUIRES HUMAN DECISION] items remain

### From Architect/Explorer to SRE/Ops
- [ ] Architecture components documented
- [ ] Technology choices grounded in constraints
- [ ] Interfaces specified (sync/async, protocols, timeouts)
- [ ] Scaling limits identified
- [ ] Dependencies listed with SLAs

### From Architect/Explorer to Cost Lead
- [ ] Infrastructure details (instance types, storage, networking)
- [ ] Expected throughput & scale
- [ ] Technology choices finalized
- [ ] Operational headcount required
- [ ] Learning timeline & budget

### From SRE/Ops to Lead Architect
- [ ] Failure modes documented
- [ ] Monitoring strategy defined
- [ ] Operational burden estimated
- [ ] [OPERABILITY_RISK] items flagged
- [ ] Runbook templates drafted

### From Cost Lead to Lead Architect
- [ ] Budget estimate with breakdown
- [ ] Cost trade-offs identified
- [ ] ROI analysis vs. alternatives
- [ ] Scaling cost model
- [ ] Budget constraints documented

### From Analyst to Lead Architect
- [ ] Assumptions register complete
- [ ] Validation plans drafted
- [ ] Evidence citations provided
- [ ] [REQUIRES_VALIDATION] items flagged
- [ ] Decision rationale documented

### From Product/Velocity to Lead Architect
- [ ] Feature delivery strategy
- [ ] Deployment coordination requirements
- [ ] Velocity impact estimated
- [ ] Rollback procedures designed
- [ ] Technical debt plan drafted

### From Security/Compliance to Lead Architect
- [ ] Data protection strategy
- [ ] Compliance checklist completed
- [ ] Access control designed
- [ ] Threat model & mitigations
- [ ] Security assumptions validated

---

## Decision Rights by Role

**Who has final say on different decisions?**

| Decision Type | Final Authority | Consulted | Informed |
|---------------|-----------------|-----------|----------|
| Technology choice | Morpheus (Lead) | Neo (Ops), Tank (Evidence) | Switch (Velocity), Trinity (Cost) |
| Operational approach | Neo (SRE) | Morpheus (architecture fit), Tank (risk) | Everyone |
| Budget allocation | Trinity (Finance) | Morpheus (priority), Neo (necessity) | Everyone |
| Compliance/Security | Agent Smith (CISO) | Morpheus (architecture), Tank (requirements) | Everyone |
| Feature strategy | Switch (Product) | Morpheus (feasibility) | Neo (ops impact), Trinity (cost impact) |
| Requirement prioritization | Tank (Analyst) | Morpheus (feasibility), Trinity (cost) | Everyone |

---

## Role Conflict Resolution

**When roles disagree, escalate per this matrix:**

| Conflict | Resolution | Authority |
|----------|------------|-----------|
| SRE says ops burden too high; Lead says architecture necessary | Negotiate simplification; if no middle ground, escalate to CTO | CTO |
| Cost Lead says budget exceeded; Lead says necessary for requirements | Renegotiate requirements with stakeholders | Executive Sponsor |
| Product wants fast iteration; SRE warns of operational risk | Design circuit breakers & monitoring; proceed with caution; flag [TEAM_REVIEW_NEEDED] | Morpheus (Lead) |
| Security finds compliance gap; timeline is tight | Security requirement takes priority; negotiate delay or scope reduction | CISO + Executive |
| Analyst flags unvalidated assumption; Lead wants to proceed | Proceed with contingency plan if assumption fails; flag as [HIGH_RISK] | Morpheus (Lead) |

---

## Role Collaboration Pattern

**Typical workflow:**

```
Day 1: Requirements Analyst (Tank) clarifies requirements + constraints
  ↓
Day 2: Lead Architect (Morpheus) sketches options; SRE (Neo) identifies ops concerns
  ↓
Day 3: Architect finalizes spec; Cost (Trinity) estimates budget; Security (Smith) reviews
  ↓
Day 4: Product (Switch) plans feature delivery; Analyst (Tank) documents assumptions
  ↓
Day 5: Lead Architect synthesizes final decision; all roles review for completeness
  ↓
Day 6: Team review; document ADRs; update decision log
```

---

## Role Staffing Models

### Model 1: Squad (Ideal)
Each role staffed by 1 person:
- Morpheus: Lead Architect
- Neo: SRE / Ops Lead
- Trinity: Finance / Program Manager
- Tank: Research / Requirements Analyst
- Switch: Product Manager / Engineering Manager
- Agent Smith: Security / Compliance Lead

**Pros:** Clear accountability; no ambiguity  
**Cons:** Requires 6 people; expensive for startups

### Model 2: Lean (Startup)
One person covers multiple roles:
- Morpheus + Switch: Tech Lead (architecture + product prioritization)
- Neo + Trinity: Whoever handles DevOps + budgets (often the CTO)
- Tank: Shared responsibility (everyone documents decisions)
- Agent Smith: Consulting (external security review as needed)

**Pros:** Lean staffing; lower cost  
**Cons:** Potential conflicts of interest; decision rationale less documented

### Model 3: AI + Humans (Hybrid)
AI agents for research / synthesis; humans for decisions:
- Morpheus: Human Lead Architect (decisions)
- Neo: AI SRE (generates runbooks, identifies risks)
- Trinity: Human Finance Lead (decisions)
- Tank: AI Analyst (extracts requirements, maintains decision log)
- Switch: Human Product Manager (decisions)
- Agent Smith: Human Security (decisions)

**Pros:** Scalable; AI handles research burden  
**Cons:** Requires AI training; human decision-makers still needed

---

## Role Skills & Training

### Morpheus (Lead Architect)
**Essential Skills:**
- Constraint-based architecture reasoning
- Technology landscape (databases, messaging, caching, compute)
- Tradeoff analysis (cost vs. complexity vs. velocity vs. reliability)
- ADR writing and decision documentation
- Stakeholder communication

**Training:** 5+ years architecture experience; mentorship in constraint-grounded reasoning

### Neo (SRE / Operations)
**Essential Skills:**
- Production systems operation
- Incident response & runbooks
- Monitoring & observability
- Failure mode analysis
- Infrastructure cost optimization

**Training:** 3+ years ops/SRE experience; knowledge of deployment platforms (Docker, Kubernetes, serverless)

### Trinity (Finance / Program)
**Essential Skills:**
- Cost estimation (infrastructure + people + learning)
- Budget management & forecasting
- ROI analysis
- Vendor negotiation
- Financial modeling for scaling

**Training:** Finance or program management background; cloud cost familiarity

### Tank (Research / Requirements)
**Essential Skills:**
- Requirements gathering & clarification
- Evidence-based reasoning
- Gap analysis
- Decision documentation
- Assumption validation planning

**Training:** Requirements engineering or business analysis background; research orientation

### Switch (Product / Velocity)
**Essential Skills:**
- Feature prioritization
- Technical debt management
- Deployment strategy
- A/B testing & experimentation
- Engineer productivity optimization

**Training:** Product management or engineering management background

### Agent Smith (Security / Compliance)
**Essential Skills:**
- Security architecture & threat modeling
- Compliance frameworks (GDPR, HIPAA, PCI, SOC2)
- Data protection & encryption
- Access control & audit logging
- Vendor security review

**Training:** Security or compliance background; certifications (CISSP, CCSK) helpful

---

## Onboarding New Team Members to Roles

**For each role, provide:**
1. Role charter (this document)
2. Success criteria (clear definition of done)
3. Templates (ADR, runbook, spec, etc.)
4. Examples (good & weak outputs)
5. Escalation paths (who to ask when stuck)
6. Tools & systems (where docs live, how to update them)

**Recommended onboarding timeline:**
- Week 1: Read charter; understand decision rights + responsibilities
- Week 2: Pair with current role holder; observe 2–3 decisions
- Week 3: Lead role holder; pair on all decisions
- Week 4: Lead independently; role holder reviews output
