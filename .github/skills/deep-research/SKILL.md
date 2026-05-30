---
name: deep-research
description: 'Web-grounded brainstorming and deep research methodology for the generic idea-to-ship agent framework. Turns a raw idea or problem statement into a cited, decision-ready dossier before PRD writing. Uses structured question decomposition, source retrieval, citation discipline, trade-off analysis, scaling strategy, and an assumption ledger. USE FOR: vague product ideas, unfamiliar domains, architecture/scaling choices, competitor/prior-art scans, and high-risk features that need external grounding before a PRD. DO NOT USE FOR: supplied PRDs that are already clear, private-code-only debugging, or implementation. OUTPUT: `docs/research/<slug>.md`, then hand off to the prd agent.'
---

# SKILL: deep-research

Use this skill before PRD authoring when the user has a raw idea, asks for brainstorming, or needs external grounding. The goal is not a long essay; it is a dense dossier an implementation workflow can use without browsing again.

Reference methodology: [`docs/research/03-deep-research-patterns.md`](../../../docs/research/03-deep-research-patterns.md).

Recommended agent: [`research`](../../agents/research.agent.md), which writes only `docs/research/*.md` and hands the dossier to [`prd`](../../agents/prd.agent.md).

---

## Inputs

Accept:

- Raw idea or problem statement.
- Target users and platforms, if known.
- Constraints: timeline, budget, privacy, compliance, preferred stack, non-goals.
- Existing links, competitors, docs, or internal notes.
- Desired depth: `quick`, `standard`, or `max`.

If depth is not specified:

- `quick` for naming/positioning or PRD sketch.
- `standard` for most product/technical planning.
- `max` for regulated, security-sensitive, scaling-heavy, or expensive-to-reverse decisions.

---

## Process

### 1. Normalize intake

Write a compact brief:

- Problem and target user.
- Job-to-be-done.
- Success metric.
- Target platforms from [`PROJECT.md`](../../../PROJECT.md), if relevant.
- Non-goals and constraints.
- Assumptions that need validation.

### 2. Decompose into research questions

Create 5–12 questions across:

- Prior art and similar projects.
- User needs and failure modes of current solutions.
- Technical feasibility and architecture options.
- Scaling, reliability, cost, and operations.
- Privacy, security, legal, and compliance risk.
- Monetization or adoption, when product-relevant.
- Platform/profile implications.

For `max` depth, allow recursive follow-up questions where evidence conflicts or risk is high.

### 3. Search and fetch

For each question:

1. Expand queries with exact terms, synonyms, competitors, open-source projects, "alternatives", "pitfalls", and version-specific terms.
2. Prefer authoritative and diverse sources: official docs, standards, GitHub repos, papers, benchmarks, changelogs, pricing pages, app listings, and credible technical writeups.
3. Fetch primary pages instead of citing search snippets.
4. Track source type, freshness, relevance, and caveats.

Stop a branch when three independent high-quality sources agree or contradictions are documented.

### 4. Extract and compress

For each useful source record:

- Claim or pattern.
- Link.
- Why it matters.
- Caveat or uncertainty.
- Confidence: high/medium/low.

Keep notes dense. Do not paste long quotes unless exact wording is critical.

### 5. Synthesize approaches

Brainstorm 2–4 viable approaches. For each:

- User value.
- Implementation cost.
- Dependency and vendor risk.
- Scaling path.
- Security/privacy risk.
- Platform/profile complexity.
- Testing and observability needs.
- Why this approach might fail.

Use a decision matrix when trade-offs are non-obvious.

### 6. Recommend

Choose a recommended default and explain:

- Why it is the best current bet.
- MVP path, v1 path, and scale path.
- What to avoid for now (anti-roadmap).
- Assumptions to validate during PRD/planning.

### 7. Citation audit

Every source-grounded claim needs an inline link. Any uncited claim must be labeled as reasoning, assumption, or recommendation.

No link should appear without a short reason it is relevant.

---

## Output dossier

Write exactly one dossier to `docs/research/<slug>.md`. Create `docs/research/` if needed. The slug should be kebab-case and stable.

Template:

```markdown
# <Title> — Research Dossier

## Problem framing
- User / buyer:
- Job-to-be-done:
- Goal metric:
- Constraints:
- Non-goals:

## Research questions
1. ...

## Prior art & similar projects
| Project/source | What it does | Relevant pattern | Caveat | Link |
|---|---|---|---|---|

## Key findings
- <claim with inline link> — <why it matters>

## Approaches & trade-offs
| Approach | Pros | Cons | Scaling path | Risk | Fit |
|---|---|---|---|---|---|

## Scaling / architecture considerations
- MVP:
- v1:
- Scale version:
- Observability / operations:

## Risks & open questions
| Risk/question | Why it matters | Validation method | Owner/phase |
|---|---|---|---|

## Assumption ledger
| Assumption | Evidence | Confidence | How to validate |
|---|---|---|---|

## Recommendation
- Recommended approach:
- Why:
- Anti-roadmap:
- Handoff notes for PRD:

## Sources
- [Source](https://example.com) — reason used, freshness if version-sensitive.
```

---

## Handoff to PRD

After writing the dossier:

1. Summarize the recommendation in 5–8 bullets.
2. Provide the dossier path.
3. Hand off to [`prd`](../../agents/prd.agent.md) with: raw idea, dossier path, assumptions, recommended approach, and open questions.
4. Do not edit application code or project configuration.

---

## Quality rubric

A good dossier is:

- **Actionable:** names concrete patterns, APIs, tests, and decisions.
- **Verifiable:** links near claims; uncertainties visible.
- **Balanced:** includes alternatives and failure modes.
- **Token-efficient:** dense tables and bullets, not narrative sprawl.
- **Decision-ready:** PRD author can proceed without re-researching.
