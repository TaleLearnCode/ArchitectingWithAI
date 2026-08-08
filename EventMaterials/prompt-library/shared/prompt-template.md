# Prompt Card Template

Use this template structure for every prompt in the library. This ensures consistency and helps attendees know what to expect.

---

## Template Structure

### Use This When
**1–2 sentences.** When should an architect reach for this prompt?
- Example: "You have requirements but they feel incomplete or contradictory. Use this to surface ambiguities before architecture work begins."

### Prerequisites
**Bulleted list.** What must be ready before this prompt works?
- Example inputs: requirements document, team size, budget constraints
- Data format: "User stories with acceptance criteria" or "Architecture diagrams in C4 format"
- Context: "Understanding of current system" or "Prior decisions captured in ADRs"

**Validation Rule:** If a prerequisite is missing, the prompt will output `[INPUT MISSING: <name>]`. Do not proceed with assumptions.

---

### Input Example

Paste a realistic sample of what attendees will provide. Keep it concrete and project-like.

```
[Example: Paste one complete user story with acceptance criteria, 
or a requirements excerpt, or a team profile—whatever THIS prompt needs]
```

**What to replace:**
- Replace `[Placeholder]` with your actual data
- Keep structure but customize content
- If you have 10 requirements but space is tight, paste 3 representative examples

---

### The Prompt

**Copy-paste ready.** This is the exact text to submit to your AI platform.

```text
[FULL PROMPT TEXT HERE]

Use three asterisks (***) on their own line to separate the prompt from your input data.
```

**Platform Note:** Works on GitHub Copilot CLI, ChatGPT (GPT-4+), Claude. Phrasing is robust across all three.

---

### Customization Knobs

**How to adjust output if the default prompt doesn't match your need.**

| Knob | What It Changes | How to Use |
|------|-----------------|-----------|
| **Depth** | Amount of detail in output | Add: "Be thorough" for deep analysis; "Be brief" for summaries |
| **Focus** | Which aspects AI prioritizes | Add: "Emphasize security implications" or "Focus on cost" |
| **Tone** | How aggressive AI is | Add: "Be diplomatic" for stakeholder-friendly; "Be brutal" for adversarial review |
| **Scope** | What AI includes/excludes | Add: "Consider only these technologies" or "Ignore infrastructure details" |
| **Format** | How output is structured | Add: "Use a table" or "Use bullet points" |

**Example modification:**
- Default prompt: "Generate 4 architecture options"
- Your twist: "Generate 4 options. Focus on cost and team velocity. Exclude serverless (compliance issue)."

---

### Example Input & Output

**Good example:** Show real-world input and what good output looks like.

```
INPUT (attendee provides):
Requirements: E-commerce platform, 500K users/month, $200K infrastructure budget, 
6-person team (no specialized DBAs), must launch in 6 months.

OUTPUT (AI provides):
[Good output that: is specific to these constraints, references them explicitly, 
avoids boilerplate, traces reasoning back to inputs]
```

**Weak example:** Show a common failure mode.

```
WEAK OUTPUT (don't accept this):
"Consider microservices because they are scalable and modern. 
Or use serverless for faster time-to-market."

WHY IT'S WEAK:
- No connection to the 500K users constraint
- No acknowledgment of "6 months to launch"
- Generic boilerplate; could apply to any project
- No reasoning about why these tradeoff matter for THIS team
```

**How to fix it:** "Add more specificity: For our 6-person team with 6 months, 
explain how each option impacts time-to-market and operational overhead."

---

### Validation Checklist

**Use this BEFORE acting on output.** Mark each item; if you have more than 1 "no," output needs iteration.

- [ ] **Traceable to input:** Every recommendation references something I provided (not invented)
- [ ] **Specific to my project:** Output changes if I change team size, budget, or constraints
- [ ] **No generic boilerplate:** Phrases like "best practices" and "highly scalable" come with context
- [ ] **Confidence grounded:** If AI rates options, ratings reference explicit constraints
- [ ] **Assumptions flagged:** Anything inferred by AI is marked `[INFERRED]` with reasoning
- [ ] **No invented technologies:** All recommendations use tech/approaches we discussed

**Scoring:**
- **All checked:** Output is trustworthy. Act on it.
- **1–2 unchecked:** Iterate. Follow up with a question.
- **3+ unchecked:** Output needs major revision or different prompt approach.

---

### Iteration Loop

**If output doesn't meet validation checklist, iterate here.**

#### Iteration 1 (Default Fails)
- Problem: [What didn't work]
- Follow-up: [Specific question or constraint to add]
- Result: [Did it improve?]

#### Iteration 2 (If Still Needed)
- Problem: [Remaining gap]
- Follow-up: [Next modification]
- Result: [Did it improve?]

#### Fallback
If iterations don't improve output, try switching AI platforms (ChatGPT → Claude) or use the related prompt in Section [X].

---

### Pro Tips

- **Timing:** Use this prompt when [specific project phase]. Doing it too early means unclear inputs; too late means rework.
- **Team collaboration:** Best run synchronously with [role/team]. Have [role] validate output before proceeding.
- **Common mistakes:** Attendees often [common error]. To avoid: [how to prevent it].
- **Scaling up:** If you're managing 20+ prompts (multi-team architecture), [advanced technique].

---

### Guardrails Applied

This prompt is designed to prevent specific hallucinations:

- **Pattern:** [Name of hallucination pattern this prevents]
- **How it prevents:** [Technical detail of how phrasing was tuned]
- **Example failure if guardrail removed:** [What would go wrong]

---

### Related Prompts

- **Use together with:** [Other Section/Prompt that chains well]
- **Use instead of:** [Alternative if this doesn't fit your need]
- **Later step:** [What prompt comes next in the workflow]

---

## Inline Template Example

Below is a filled-in example of what a real prompt card looks like.

---

## ✓ Requirements Ambiguity Scan (EXAMPLE)

### Use This When
Your team has written user stories and acceptance criteria, but you suspect they're incomplete or contradictory. Use this to surface ambiguities, implicit requirements, and hidden constraints before spending time on architecture.

### Prerequisites
- User stories with acceptance criteria (minimum 5 stories; representative of the problem)
- List of known constraints (team size, budget, SLA, compliance)
- Context about the business (what problem are we solving?)

**Validation Rule:** If user stories are missing, output will say `[INPUT MISSING: User Stories]`. Gather them first.

---

### Input Example

```
USER STORIES:

Story 1: As a merchant, I want to upload product catalog so that customers can browse items.
Acceptance Criteria:
- Supports CSV, XML, JSON formats
- Validates data and reports errors
- Updates live catalog within 2 hours

Story 2: As a customer, I want to search products by category so that I can find what I need.
Acceptance Criteria:
- Search must complete in < 1 second
- Results must include at least 10 facets (size, color, brand, etc.)

KNOWN CONSTRAINTS:
- Team: 3 backend engineers, 2 frontend, 1 QA
- Budget: $100K/year
- Regulatory: GDPR, SOC2
```

---

### The Prompt

```text
Analyze these user stories and acceptance criteria. Identify:

1. Ambiguities (what's unclear?)
2. Contradictions (what conflicts?)
3. Missing acceptance criteria (what's implied but not stated?)
4. Implicit non-functional requirements (performance, security, reliability implied by the stories?)
5. Hidden constraints (what does the business model imply?)
6. Assumptions (what must be true for these stories to work?)
7. Clarifying questions (what must the team answer before designing?)

For each finding, explain why it matters for architecture.

Do NOT invent new features or stories. Only analyze what is present or reasonably implied by the statements above.

***
[PASTE USER STORIES AND CONSTRAINTS HERE]
```

---

### Customization Knobs

| Knob | Use When | Example |
|------|----------|---------|
| **Depth** | You want deep analysis | Add: "For each ambiguity, provide 2–3 clarifying questions the architect should ask" |
| **Focus** | You care about specific risks | Add: "Focus on performance, security, and data consistency" |
| **Stakeholder angle** | You want perspective from a role | Add: "Answer from the perspective of an SRE: What reliability risks do you see?" |
| **Format** | You want structured output | Add: "Use a table with columns: Finding Type, Description, Impact on Architecture" |

---

### Validation Checklist

- [ ] **Traceable:** Every finding references the user stories I provided
- [ ] **Specific:** Ambiguities name the exact phrase that's unclear (not just "requirements are vague")
- [ ] **No invention:** Implied NFRs come from obvious clues in the stories (not AI guessing)
- [ ] **Actionable:** Each clarifying question is something the team can answer (not philosophical)
- [ ] **No boilerplate:** Output doesn't just say "consider security and reliability"—it names specific gaps

---

### Example Output (Good)

```
FINDING 1: Ambiguity in "Updates live catalog within 2 hours"
- Exact phrase: "Updates live catalog within 2 hours"
- Why it's ambiguous: Does this mean all customers see the update within 2 hours? 
  Or does the batch process complete in 2 hours (but customers see it later)?
- Impact on architecture: This determines whether we use eventual consistency or 
  require strong consistency. Microservices vs. monolith tradeoff depends on this.
- Clarifying question: "When a merchant uploads a product, what's the maximum acceptable 
  delay before a customer can see it in search results?"

IMPLIED NFR 1: High search performance
- Evidence: "Search must complete in < 1 second" + "Results include 10+ facets"
- Implication: Search is a performance-critical path. This favors read-optimized 
  architectures (caching, denormalization, search indices).
- Design consequence: Consider whether a standard RDBMS is sufficient or if you need 
  Elasticsearch or similar.

HIDDEN CONSTRAINT: Batch processing is acceptable
- Evidence: Merchant uploads trigger offline processing ("2 hour" SLA), not real-time
- Implication: This is NOT a real-time event streaming system. Batch architecture is fine.
- Design consequence: Simpler than you might think; reduces the need for Kafka, complex workflows.
```

---

### Iteration Loop Example

If initial output is too generic:

**Iteration 1:**
- Problem: AI said "requires high availability" but didn't explain what that means here
- Follow-up: "The team is 5 engineers total. For each NFR you identified, explain what it costs (infrastructure, complexity, team effort) and ask: Is it worth it given our team size?"
- Result: Output became specific; suggested accepting 2–4 hour maintenance windows instead of 99.99% uptime

---

## End of Template

Use this structure for every prompt you create. Consistency makes the library trustworthy and easy for attendees to navigate.
