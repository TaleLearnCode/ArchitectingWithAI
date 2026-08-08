# Shared Resources — Prompt Library

Common templates, instructions, and reference materials used across all sections.

## Purpose

This folder contains reusable assets that support the prompts and guidance across the library:
- Template instructions (ADRs, process flows, specifications, risk registers, assumptions)
- Shared personas and stakeholder profiles
- Common constraint vocabularies
- Validation checklists and quality gates
- Reference materials (ISO 25010, architectural patterns, etc.)

## Contents

### Template Instructions
Standardized instructions for structured output — used by Section 4 (Documentation) and embedded in prompts across other sections:
- `adr.instructions.md` — Architecture Decision Record template instructions
- `process-flow.instructions.md` — Process Flow template instructions
- `architecture-specifications.instructions.md` — Architecture Specifications template
- `risk-document.instructions.md` — Risk Register Entry template
- `assumption.instructions.md` — Assumption Register Entry template

### Personas & Stakeholders
Consistent profiles used in requirements analysis and adversarial review:
- `personas.md` — Architect, requirements analyst, SRE, security, compliance, CFO, end user profiles
- `stakeholder-perspectives.md` — Role-specific concerns and quality attributes each stakeholder prioritizes

### Constraints & Vocabulary
Shared reference for consistent constraint language:
- `constraints-vocabulary.md` — Standard constraint categories (team size, deployment, regulatory, latency, budget, operational maturity)
- `quality-attributes.md` — ISO 25010 model and attribute definitions

### Validation & Quality Gates
Checklists and rubrics for evaluating AI outputs:
- `quality-gates.md` — Checklist for hallucination detection, specificity verification, grounding validation
- `anti-slop-checklist.md` — Warning signs (generic filler, invented capabilities, ungrounded claims, repeated content)
- `constraint-grounding-rubric.md` — How to verify that architecture recommendations respect stated constraints

## How to Use This Folder

1. **In prompts:** Reference template instructions by name (e.g., `[Paste adr.instructions.md]`)
2. **In guidance:** Link to personas and vocabularies when explaining customization
3. **In examples:** Use shared personas and constraints to make examples reproducible
4. **In validation:** Apply shared quality gates across all output — consistency matters

## Ownership & Updates

This folder is owned collectively by the Squad. Updates should:
- Maintain backward compatibility with all six sections
- Be validated against at least two example prompts before merge
- Be documented in `.squad/decisions/` if the change affects multiple sections

---

**Last Updated:** 2026-06-01 (Neo — Initial Scaffold)
