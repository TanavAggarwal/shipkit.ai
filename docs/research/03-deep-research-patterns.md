# Deep-research & brainstorming agent patterns

Purpose: reusable patterns for a generic publishable `idea -> ship` framework whose agents can turn a vague idea into a web-grounded PRD, technical research brief, and implementation context.

## Source landscape

| System | Public pattern to copy | Framework implication |
|---|---|---|
| OpenAI Deep Research | Autonomous multi-step web research, browsing/analysis of text/images/PDFs, adaptive plan, cited report; still needs human verification for high-stakes claims ([OpenAI](https://openai.com/index/introducing-deep-research/)). | Treat research as long-running, auditable work with citations and explicit uncertainty, not a single answer. |
| Gemini Deep Research | API agent that plans, executes, and synthesizes multi-step research; tasks take minutes; supports `background=true`, polling, collaborative planning, MCP tools, document inputs, Deep Research vs Max modes ([Google](https://ai.google.dev/gemini-api/docs/interactions/deep-research)). | Add async research jobs, plan-review before execution, source/tool connectors, and depth presets. |
| GPT-Researcher | Planner generates research questions; execution/crawler agents gather info; publisher aggregates report; tracks sources; supports web/local docs, MCP, deep recursive tree search with configurable breadth/depth ([GitHub](https://github.com/assafelovic/gpt-researcher)). | Use planner -> parallel retrievers -> context compressor -> writer pipeline. |
| STORM / Co-STORM | Pre-writing research gathers references and outline; perspective-guided question asking; simulated writer/expert conversations grounded in search; Co-STORM adds moderator, experts, user, and mind map ([Stanford OVAL](https://github.com/stanford-oval/storm)). | For brainstorming, force multiple perspectives and outline-first synthesis before PRD. |
| Perplexity-style answer engines | Query refinement, source selection, inline citations, synthesis of source agreement/disagreement ([Perplexity Pro Search blog](https://www.perplexity.ai/blog/pro-search)). | Show sources next to claims; separate “known”, “inferred”, “unknown”. |

## Canonical deep-research loop

1. **Intake**: capture idea, target user, platform, constraints, business goal, non-goals, risk tolerance.
2. **Research plan**: generate 5-12 subquestions across market, users, competitors, technical feasibility, risks, monetization, legal/privacy, and implementation.
3. **Plan review gate**: present/edit subquestions before spending tokens/time; allow “fast”, “standard”, “max” depth.
4. **Search query expansion**: for each subquestion, generate exact-keyword, synonym, competitor, academic, open-source, “alternatives”, and “pitfalls” queries.
5. **Parallel retrieval**: fetch diverse sources: official docs, GitHub repos, docs, papers, benchmarks, blog posts, app listings, standards, pricing pages.
6. **Source triage**: rank by authority, recency, specificity, independence, and evidence type; demote SEO summaries unless corroborated.
7. **Per-source notes**: extract claim, quote/summary, URL, date if visible, relevance, caveat, and source type.
8. **Contradiction pass**: identify conflicts, stale docs, vendor claims, missing evidence, regional/platform differences.
9. **Synthesis**: produce dense context files, not chat prose: decisions, trade-offs, open questions, implementation patterns, commands, links.
10. **Reader test**: ask “Could an implementation agent act without browsing again?” If no, add concrete APIs, repo names, setup commands, examples.
11. **Citation audit**: every external claim gets an inline link; uncited claims are labeled as reasoning/assumption.
12. **Handoff**: store artifacts in `docs/research/` and/or task context; summarize sources and risks in PRD/plan.

## Research artifact types

| Artifact | When | Contents | Size target |
|---|---:|---|---:|
| `market-scan.md` | Idea validation | competitors, positioning, pricing, unmet needs, target users | 100-250 lines |
| `technical-options.md` | Architecture choices | option table, constraints, recommended default, migration path | 100-300 lines |
| `risk-register.md` | Complex domains | privacy/security/legal/model risk, mitigations, tests | 50-150 lines |
| `implementation-context.md` | Before coding | exact libraries, commands, APIs, examples, gotchas | 150-400 lines |
| `source-map.md` | Auditing | source list, reason used, freshness, confidence | 50-200 lines |

## Brainstorming patterns

- **Ground before ideating**: first fetch similar products/projects; then ideate against real patterns, not model priors.
- **Analogy matrix**: rows = adjacent tools; columns = onboarding, core loop, moat, pricing, failure modes, technical stack.
- **User-pressure test**: for each idea, list top 3 urgent jobs-to-be-done and why current tools fail.
- **Differentiation test**: “If a clone of X exists, why would anyone switch?” Require one clear wedge.
- **Feasibility gradient**: MVP in 1 week, v1 in 1 month, scale version in 6 months.
- **Anti-roadmap**: explicitly list attractive features to avoid until signal exists.
- **Assumption ledger**: each key assumption gets validation method: source, prototype, user interview, benchmark, or test.
- **Decision matrix**: score options on user value, implementation cost, dependency risk, cross-platform cost, privacy/security risk.

## Source gathering & citation discipline

- Prefer official docs for APIs/tool behavior (e.g., Gemini Deep Research docs for background execution and collaborative planning).
- Prefer GitHub README/docs for open-source architecture (e.g., GPT-Researcher planner/execution/publisher, STORM pre-writing/writing stages).
- Use secondary articles for product capabilities only when official sources are inaccessible; mark lower confidence.
- Store citations inline near claims, not as a dump at the end.
- Avoid citing search-result snippets as facts when a direct page can be fetched.
- Record source freshness when version-sensitive: “Expo SDK 56”, “Playwright Node 20/22/24”, “OpenHands 1.7 docs”.
- For generated context files, use compact footnote-free links: `[tool](url)`.

## “Think long/deep” without wasting context

- Use **externalized state**: research plan, source map, notes, and synthesis files on disk; agent context only loads relevant sections.
- Use **breadth/depth budgets**: e.g., breadth 8 subquestions x depth 5 sources; max mode 15 x 10.
- Use **recursive deepening** only where uncertainty matters: market size, regulated domains, hard technical unknowns.
- Compress after each branch: `facts`, `trade-offs`, `evidence`, `uncertainties`, `next queries`.
- Ask the model to produce “decision-ready summaries” rather than exhaustive summaries.
- Use a “stop rule”: stop researching a subquestion after 3 independent high-quality sources agree or after contradictions are documented.

## Research loop pseudocode

```text
research(task, depth):
  brief = normalize_intake(task)
  plan = generate_subquestions(brief, depth)
  if collaborative: approve_or_refine(plan)
  for q in parallel(plan.questions):
    queries = expand(q)
    docs = retrieve_and_fetch(queries, source_policy)
    notes[q] = extract_claims(docs)
    gaps[q] = contradictions_and_unknowns(notes[q])
    if depth == max and gaps[q].material: recurse(q, gaps[q])
  synthesis = write_context_files(brief, notes, gaps)
  audit_citations(synthesis)
  return synthesis
```

## Evaluation rubric for research outputs

| Criterion | Good | Bad |
|---|---|---|
| Actionability | Names libraries, commands, files, APIs, tests | Generic advice |
| Verifiability | Inline links per claim; caveats visible | Uncited assertions |
| Breadth | Covers market, users, tech, risks, alternatives | Only confirms preferred path |
| Depth | Explains why choices matter and failure modes | Feature list only |
| Token efficiency | Dense tables + bullets; no long quotes | Narrative essay |
| Handoff quality | Implementation agent can proceed offline | Requires re-research |

## Lessons for our framework

1. Add a **`research` phase before PRD finalization**: idea -> source-grounded brainstorming -> PRD, not idea -> PRD.
2. Make research **async and resumable**: save plan, per-source notes, source map, synthesis, and status; long jobs may take minutes ([Gemini docs](https://ai.google.dev/gemini-api/docs/interactions/deep-research)).
3. Support **collaborative planning** even in autonomous mode: agent writes a proposed plan, then self-reviews it against scope before execution.
4. Implement **planner / retriever / synthesizer / citation-auditor roles**; GPT-Researcher shows this decomposition is practical ([GPT-Researcher](https://github.com/assafelovic/gpt-researcher)).
5. For brainstorming, copy STORM’s **perspective-guided questions**: competitors, users, operators, security, platform, cost, compliance, growth ([STORM](https://github.com/stanford-oval/storm)).
6. Require all research outputs to include: `Key findings`, `Trade-offs`, `Recommended default`, `Open questions`, `Sources`, and `Lessons for our framework`.
7. Build a **citation gate**: no source-grounded claim without a link; no link without a short reason it is relevant.
8. Add **depth presets**: `quick` for PRD sketch, `standard` for build planning, `max` for scaling/security/regulated features.
9. Favor **context files over long chat** so implementation agents can load only what matters.
10. Always output an **assumption ledger** that later dev/test agents can convert into verification gates.
