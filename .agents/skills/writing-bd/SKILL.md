---
name: writing-bd
description: "Skill for authoring Basic Design (BD) documents at Stage 3 of the Senior Workflow. Use when you need to align stakeholders (PO, PM, tech lead) on WHAT and WHY of a feature BEFORE investing effort in HOW (which is DDD's job). BD audience = non-deeply-technical readers. Required structure: Header (status/owner/stakeholders/impact), Background & Why with measurable goals, Goals & Non-goals, User Stories with acceptance criteria, High-level Solution with a Mermaid flowchart at component/actor level, Affected Components table, Alternatives Considered with rejection reasons, Open Questions, Assumptions, Decision Log. BD is tri-lingual (EN canonical + VI + JP). Status starts as DRAFT, never set FINAL here. Length target: 1–3 pages."
---

# Skill: Writing Basic Design (BD)

## When to use
- Stage 3 of the workflow, before authoring DDD.
- When you need to align stakeholders on **what** and **why** before investing effort on **how**.

## Purpose
- **Audience**: PO, PM, tech lead, other devs (not necessarily deep-technical).
- **Answers**: Why are we doing this? What are we doing? Who uses it? What is the high-level shape?
- **Does NOT answer**: API schema, SQL, code, detailed sequence diagrams → that's DDD.

## Required structure

### 1. Header
```yaml
Title: <feature name>
Status: DRAFT | REVIEW | FINAL
Owner: <author>
Stakeholders: <PO, tech lead, ...>
Linked task: <Jira/Linear link>
Impact: 🟢/🟡/🟠/🔴
Created: <YYYY-MM-DD>
```

### 2. Background & Why
- **Current situation**: user/business pain points.
- **Business goal**: measurable (e.g. reduce 30% support tickets about X).
- **Why now**: priority justification.

### 3. Goals & Non-goals
- **Goals**: 3–5 measurable outcomes.
- **Non-goals**: explicit list of things we are NOT doing this round.

### 4. User Stories
Format: `As a <role>, I want <action>, so that <benefit>`.
Each story has high-level acceptance criteria (details go in DDD).

### 5. High-level Solution
- 2–3 paragraphs describing the approach.
- **Mandatory diagram**: 1 Mermaid `flowchart` at component/actor level showing the end-to-end processing flow.

### 6. Affected Components
Table listing modules/services affected + change type (new / modify / breaking).

### 7. Alternatives Considered
≥2 alternatives with reasons NOT chosen — critical to avoid re-litigation later.

### 8. Open Questions
Numbered questions for stakeholders. When all are answered → can move to DDD.

### 9. Decision Log
| Date | Decision | Rationale | Decided by |
|---|---|---|---|

## Writing rules
- 1–3 pages is enough. More → may be drifting into DDD territory.
- Business-friendly language, avoid deep technical jargon.
- Every decision has a rationale.
- All assumptions explicit in `Assumptions`.
- Tri-lingual: EN canonical + VI + JP using collapsible blocks per section.

## Anti-patterns
- BD containing SQL / code → push to DDD.
- BD without measurable goals → return to PO.
- BD copy-pasted from task description → must synthesize.
- BD without Non-goals → invites scope creep.
