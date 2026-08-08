# How to Use This Prompt Library

Welcome. This library is a workshop-ready toolkit—not a reference manual. Think of it as a coaching guide for using AI to make architecture decisions faster and better.

## Before You Start

**You'll get the most value if:**
- You have a real project you're working on (or one you're familiar with)
- You have architectural decisions to make (or want to strengthen decisions you've already made)
- You're open to challenging your assumptions with AI as a thinking partner

**This library is NOT:**
- A substitute for domain knowledge
- A guarantee that AI won't hallucinate (it will; we show you how to catch it)
- A replacement for your own judgment (AI broadens options, you decide)

---

## How to Tailor Prompts to Your Project

Every prompt in this library includes a **Customization** section. Here's how to use it:

### The Fast Path (15–20 minutes)
1. Find the section matching your current phase (clarify, explore, review, document)
2. Copy the prompt
3. Paste your project context (requirements, team size, constraints)
4. Run it in your AI platform (GitHub Copilot CLI recommended; ChatGPT or Claude work too)
5. Validate the output using the **Validation Checklist**

### The Full Workflow (1–2 hours)
1. Start with **Section 1: Requirements Clarification**—use the "Ambiguity Scan" to surface gaps
2. Move to **Section 2: Architecture Exploration**—propose and compare options
3. Run **Section 3: Adversarial Review**—let AI attack your assumptions
4. Generate **Section 4: Documentation**—ADRs, process flows, risk registers
5. Use **Section 6: Governance**—anti-slop checks to clean up output

---

## Key Customization Patterns

### 1. Replace Template Placeholders
Every prompt has placeholders in `[BRACKETS]`. Replace them with your actual data:
- `[Your Requirements]` → Paste actual user stories
- `[Team Size]` → e.g., "6 engineers, 2 DBAs"
- `[Constraint: Budget]` → e.g., "$150K annual infrastructure"

### 2. Add Context to Narrow Output
The more specific you are, the less AI will hallucinate. Example:

**Generic:** "Generate architecture options for a e-commerce platform"  
**Better:** "Generate options for a B2B SaaS e-commerce platform: <10M revenue/year, 5-person team, must integrate with SAP, no serverless allowed due to compliance"

### 3. Iterate If Output Is Too Vague or Too Specific
- **Too vague?** Add: "Be specific about data flows and integration points."
- **Too much detail?** Add: "Focus only on the top-level component architecture, not infrastructure."

---

## Understanding Prompt Fields

Each prompt card has these sections:

| Field | What It Means | Your Job |
|-------|---------------|----------|
| **Use This When** | The decision point or problem this solves | Check: "Is this my current problem?" |
| **Prerequisites** | What you need before running the prompt | Check: "Do I have these inputs?" |
| **Input Example** | Sample data showing what to paste | Copy the structure, use your data |
| **The Prompt** | Copy-paste ready text | Copy into AI platform |
| **Customization Knobs** | Ways to adjust output | Skip if default is good; twist if you need something different |
| **Validation Checklist** | How to spot hallucinations | Run these checks before acting on output |

---

## Spotting AI Hallucinations (Critical)

AI will confidently invent things. Here's how to catch it:

### Red Flags in Output
- **Invented constraints:** "Your requirements imply you need microservices" (check: did you say that?)
- **Confident nonsense:** "This technology will handle 1M concurrent users" (check: is that based on your inputs or AI imagination?)
- **Generic boilerplate:** "Consider security, reliability, and performance" (check: did AI say anything specific?)
- **Missing evidence:** Claims with no "because" (good output traces back to your inputs)

### How to Validate
1. **Traceability:** Every claim should link back to something you said. If you didn't mention microservices, and AI recommends them, ask: "Why?"
2. **Specificity:** "This architecture fits your 5-person team" is better than "This architecture is good." Does AI reference YOUR constraints?
3. **Confidence Markers:** Look for phrases like "[INFERRED]" or "[SPECULATIVE]"—those mean AI is guessing.

---

## Common Workflows

### Workflow A: Quick Architecture Review (30 minutes)
1. Section 1: Requirements Ambiguity Scan → surface gaps
2. Section 2: Generate Options + Tradeoff Table → compare
3. Section 6: Anti-Slop Pass → clean up documentation
**Output:** A clean option comparison you can discuss with your team

### Workflow B: Full Architecture Package (2 hours)
1. Section 1: Requirements Deep Dive + Stakeholder Simulation → expose all constraints
2. Section 2: Architecture Exploration → generate and compare options
3. Section 3: Adversarial Review + Catastrophic Failure Analysis → break assumptions
4. Section 4: Generate ADRs, Process Flows, Risk Register → document decisions
5. Section 6: Governance Pass → remove slop, tighten language
**Output:** Presentation-ready architecture documentation and risk mitigations

### Workflow C: Strengthen an Existing Decision (45 minutes)
1. Section 3: Adversarial Review on your current design → expose blind spots
2. Section 2: Tradeoff Table → compare against alternatives you might have missed
3. Section 6: Quality Gate Validation → ensure your decision is grounded
**Output:** Confidence boost + list of risks to mitigate

---

## Pro Tips

1. **Use GitHub Copilot CLI** — It has context awareness; you can reference your codebase as you iterate prompts.
2. **Copy the prompt exactly** — Don't paraphrase; the phrasing is tuned to avoid hallucinations.
3. **Read the Validation Checklist first** — Before running a prompt, know what "good output" looks like.
4. **Iterate, don't just accept** — If output feels generic, ask follow-up questions. "What does this mean for my 5-person team?"
5. **Log your assumptions** — Each prompt should surface assumptions. Write them down; validate them later.
6. **Use one prompt per decision** — Don't try to run all architecture prompts in one go. Focus on one decision, get clarity, then move forward.

---

## Troubleshooting

### "AI is giving me the same 4 options for every problem"
This means the prompt is too generic. Add your specific constraints (team size, budget, regulations, SLA). The more grounded you are, the better AI output becomes.

### "Output has a lot of words but says nothing specific"
This is hallucination. Use the Anti-Slop prompt (Section 6) to cut boilerplate. Or re-run with: "Be specific. Reference the constraints I provided."

### "AI says something confident that sounds wrong"
Trust your gut. Use the Validation Checklist: Can you trace this back to your inputs? If not, mark it [SPECULATIVE] and ask a follow-up.

### "I don't know if AI output is good enough to use"
Use the Validation Checklist. If you hit 3+ "no" checkmarks, output isn't ready. Iterate or get a peer to review.

---

## Taking It Home

After the workshop:

1. **Pick one decision** from your real project
2. **Run the full workflow** (2 hours) or fast path (20 min)
3. **Compare AI output to your current approach** — Do you have blind spots? Better options?
4. **Share with a peer** — "Did I catch the hallucinations?"
5. **Document your decision** using the ADR template from Section 4

The goal: AI is a thinking partner, not a decision-maker. You stay in control. You validate. You decide.

---

## Questions?

- **How do I know which prompt to use first?** → Start with Section 1 (Requirements Clarification). You can't architect well with unclear requirements.
- **Can I skip sections?** → Yes. Use only the sections matching your current problem. You don't need all six.
- **What if the AI platform fails?** → Try a different model (ChatGPT, Claude). Phrasing is robust across platforms.
- **Can I modify the prompts?** → Yes, but carefully. The phrasing prevents specific hallucinations. If you change it, re-test output.

Enjoy. Architecture is clearer when you think with a partner. This library helps AI be a good partner.
