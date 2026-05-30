# shipkit

**A multi-agent "idea → ship" starter template.**

A clone-and-go **starter repository** for building software with AI coding agents — from a raw idea, through web-grounded research and a PRD, to a reviewed, tested, shipped implementation on **web** or **cross-platform web + iOS + Android**.

The framework is **stack-agnostic**. Your project's specifics (commands, package layout, platform rules, system shape) live in a few committed files. The agents, skills, and the workflow that binds them are reusable as-is.

> **What this is:** a portable set of [agents](.github/agents/), [skills](.github/skills/), and project conventions that turn a chat-based coding agent into a disciplined, resumable, self-verifying software team.
>
> **What this is not:** a runtime, a model, or an app. There is no product code here — only the process and the guardrails.

---

## Why it exists

Chat agents are good at writing code and bad at staying honest, staying scoped, and remembering what they were doing. This template fixes that with three ideas drawn from the current state of the art (see [`docs/research/`](docs/research/)):

1. **Phases are enforced by tools, not vibes.** A reviewer agent with no edit tools physically cannot "fix" what it reviews. A planning agent cannot touch product code. The boundary is the toolset.
2. **State lives on disk, so work survives interruptions.** Every non-trivial task persists to `.dev/tasks/<slug>/` — PRD, plan, status, journal, and a machine-readable `state.json`. A new session resumes exactly where the last one stopped.
3. **Claims must be backed by evidence.** Verification gates are declared per task and run for real; an anti-sycophancy **honesty audit** blocks "done" until command output, files, and docs actually match the claims.

---

## The agent chain

```
 research  →   prd   →   dev   →  dev-impl  ⇄  dev-review
 (idea →       (brief/    (Steps    (Step 4:     (Steps 5 + 5.5:
  cited        dossier →   0–3:      implement    gates, 3 blind
  dossier)     PRD)        plan)     only)        personas, audit)
```

| Agent | Tools it holds | Phase & job |
|---|---|---|
| [`research`](.github/agents/research.agent.md) | search + fetch + **write only `docs/research/`** | Turn a raw idea into a cited dossier on prior art, options, scaling, and risks. |
| [`prd`](.github/agents/prd.agent.md) | edit docs + search + fetch (**no app code**) | Co-author a concrete, reviewable PRD via [`doc-coauthoring`](.github/skills/doc-coauthoring/SKILL.md). |
| [`dev`](.github/agents/dev.agent.md) | full | Orchestrate Steps 0–3: resume/intake, understand, classify, plan, plan-review. |
| [`dev-impl`](.github/agents/dev-impl.agent.md) | edit + search + terminal | Step 4: implement strictly to the approved plan. |
| [`dev-review`](.github/agents/dev-review.agent.md) | **no edit** — read + terminal | Steps 5 + 5.5: run gates, three blind personas, honesty audit. Hand findings back. |

The absence of edit tools in `dev-review` *is* the review gate. The typical flow is `research → prd → dev → dev-impl ⇄ dev-review`. If you already have a PRD, start at `dev`; if you only have an idea, start at `research` (or `prd` if no web research is needed).

---

## The workflow

The master process lives in [`development-workflow`](.github/skills/development-workflow/SKILL.md). It is stack-agnostic and resumable:

```
(research?) → 0 resume│PRD-intake│research-intake
           → 1 understand → 1.5 classify → 2 plan → 3 plan-review
           → 4 implement ⇄ 5 code-review → 5.5 honesty-audit → 6 hand-off
```

- **Step 1.5 — classify** routes work as `trivial / simple / standard / complex` and skips steps that aren't worth their cost.
- **Steps 4–5 — RARV loop**: Reason → Act → Reflect → Verify. Failures loop back to implementation; only verified work advances.
- **Step 5 — code review** runs the declared verification gates, then three **blind personas**: stack/platform, security, and test-quality.
- **Step 5.5 — honesty audit** is the anti-sycophancy gate: no task is `done` until evidence matches claims.

---

## Quickstart

1. **Use this repo as a template** (GitHub → *Use this template*, or `git clone https://github.com/TanavAggarwal/shipkit.ai`).
2. **Fill in [`PROJECT.md`](PROJECT.md).** This is the *only* file required for first use. The YAML front-matter block is machine-readable and tells agents your package paths, dependency policy, and **verification gates**:
   ```yaml
   ---
   project: my-app
   target_platforms: [web]          # web | web,ios,android | backend-only
   stack_profile: docs/stack-profiles/web-spa.md
   verification_gates:
     - { name: typecheck, cwd: frontend, command: "npm run typecheck", pass: "exit 0" }
     - { name: unit,      cwd: frontend, command: "npm test -- --ci",   pass: "exit 0" }
   ---
   ```
3. **Pick a stack profile** in `PROJECT.md → stack_profile` (see the table below).
4. *(Optional)* **Grow [`ARCHITECTURE.md`](ARCHITECTURE.md)** from [`ARCHITECTURE.template.md`](ARCHITECTURE.template.md) as your system takes shape. It ships as a stub, so a fresh clone never blocks on it.
5. **Talk to an agent.** Open the [`research`](.github/agents/research.agent.md) agent with an idea, or hand a PRD straight to [`dev`](.github/agents/dev.agent.md). The workflow takes it from there.

### Pick your stack profile

| You're building… | Profile |
|---|---|
| A browser app, dashboard, SaaS UI, or web-only product | [`web-spa`](docs/stack-profiles/web-spa.md) *(default)* |
| One app for web + iOS + Android from a shared codebase | [`expo-cross-platform`](docs/stack-profiles/expo-cross-platform.md) |
| An API, worker, CLI, or backend-only service | [`backend-service`](docs/stack-profiles/backend-service.md) |

Profiles are checklist-style rule packs (forbidden APIs, required tests, example gates). They keep the framework generic while giving reviewers concrete grounds to block unsafe code. [Add your own](docs/stack-profiles/README.md#how-to-add-a-custom-profile) by copying the closest one.

---

## Autonomous dev/test harness

Agents can verify their own work end-to-end on a local server. [`app-testing`](.github/skills/app-testing/SKILL.md) boots your app, drives it with Playwright, and tears it down by PID — orchestrated by [`scripts/with_server.py`](.github/skills/app-testing/scripts/with_server.py), which takes any `--server` command and `--port`.

Dependency autonomy is governed by `PROJECT.md → dependency_policy`:

- **Dev/test tooling** (e.g. Playwright) may be installed autonomously.
- **New runtime dependencies** require a plan entry, a justification, and a reviewable lockfile diff — never silent.
- Secrets use `.env.example` placeholders; production endpoints are denied unless explicit.

These patterns are distilled from OpenHands, SWE-agent, Playwright, and others — see [`docs/research/04-autonomous-dev-test-harness.md`](docs/research/04-autonomous-dev-test-harness.md).

### Two browser paths

| Path | Use for |
|---|---|
| [`app-testing`](.github/skills/app-testing/SKILL.md) — Python Playwright + `with_server.py` | **Default.** Deterministic, committed E2E smoke tests that back verification gates and run in CI. |
| **Playwright MCP** — shipped in [`.mcp.json`](.mcp.json) | Exploratory / self-healing loops: persistent browser, live page introspection, selector discovery while debugging. |

Explore with MCP, then **codify** the result as a deterministic `app-testing` script a gate can run unattended. See [`docs/mcp.md`](docs/mcp.md) for MCP servers, registration (Copilot CLI `/mcp`), and security notes.

---

## Quality bar

The always-on rules every agent follows live in [`.github/copilot-instructions.md`](.github/copilot-instructions.md) and are enforced at the plan, review, and audit gates:

- **Structured & modular** — code is organized by feature/domain, one responsibility per module, layers (UI / logic / data) kept separate, dependencies pointing inward, new features in their own directory behind a narrow public entry point.
- **No slop** — no dead/commented-out code, no duplication, no placeholder names, no ownerless TODOs; the diff contains only what the task needs.
- **Full unit-test coverage** — every unit of non-trivial logic has a test; a `coverage` gate enforces a threshold (default ≥ 80% on logic) with captured tool output, not vibes.

---

## Skill catalog

Thirteen skills under [`.github/skills/`](.github/skills/). Workflow skills are loaded by the agents on demand (progressive disclosure); utility skills are available when relevant.

**Core workflow**

| Skill | Role |
|---|---|
| [`development-workflow`](.github/skills/development-workflow/SKILL.md) | Master process; coordinates every step below. |
| [`task-state`](.github/skills/task-state/SKILL.md) | Resumable `.dev/tasks/<slug>/` state, PRD intake/sync, status, journal. |
| [`complexity-classify`](.github/skills/complexity-classify/SKILL.md) | Step 1.5 — route trivial / simple / standard / complex. |
| [`plan-task`](.github/skills/plan-task/SKILL.md) | Step 2 — the 11-section implementation plan with gates. |
| [`review-changes`](.github/skills/review-changes/SKILL.md) | Steps 3 & 5 — plan-review and code-review with three blind personas. |
| [`honesty-audit`](.github/skills/honesty-audit/SKILL.md) | Step 5.5 — anti-sycophancy evidence gate before `done`. |

**Research, product & test**

| Skill | Role |
|---|---|
| [`deep-research`](.github/skills/deep-research/SKILL.md) | Web-grounded brainstorming → cited `docs/research/<slug>.md` dossier. |
| [`doc-coauthoring`](.github/skills/doc-coauthoring/SKILL.md) | Structured co-authoring for PRDs, specs, and decision docs. |
| [`app-testing`](.github/skills/app-testing/SKILL.md) | Local dev/test harness; self-test the running app via Playwright. |

**Utilities**

| Skill | Role |
|---|---|
| [`frontend-design`](.github/skills/frontend-design/SKILL.md) | Distinctive, production-grade UI design. |
| [`web-artifacts-builder`](.github/skills/web-artifacts-builder/SKILL.md) | Build multi-component HTML/React artifacts. |
| [`changelog-generator`](.github/skills/changelog-generator/SKILL.md) | Turn git history into user-facing release notes. |
| [`domain-name-brainstormer`](.github/skills/domain-name-brainstormer/SKILL.md) | Brainstorm and check domain-name availability. |

---

## Memory model

Three tiers — use the right one:

| Tier | Path | Holds | Lifetime |
|---|---|---|---|
| **Episodic** (per task) | `.dev/tasks/<slug>/` *(git-ignored)* | `prd.md`, `state.json`, `plan.md`, `STATUS.md`, `journal.md` | Until you delete it after merge |
| **Procedural** (per repo) | `/memories/repo/` | Verified commands, working test setups, repo gotchas | Across tasks in this repo |
| **Semantic** (cross-repo) | `/memories/` | Portable patterns — no secrets, no repo specifics | Across all repos |

---

## Repository layout

```
.
├─ README.md                     ← you are here
├─ PROJECT.md                    ← REQUIRED: your commands, packages, verification gates (YAML)
├─ ARCHITECTURE.md               ← system shape (starts as a stub)
├─ ARCHITECTURE.template.md      ← fuller template to grow ARCHITECTURE.md from
├─ .mcp.json                     ← MCP servers (Playwright) for clients that auto-load it
├─ .github/
│  ├─ copilot-instructions.md    ← always-on rules for every agent session
│  ├─ agents/                    ← research, prd, dev, dev-impl, dev-review
│  └─ skills/                    ← 13 skills (see catalog above)
└─ docs/
   ├─ mcp.md                     ← MCP server registration + security notes
   ├─ stack-profiles/            ← web-spa · expo-cross-platform · backend-service
   └─ research/                  ← the research that shaped this template
```

---

## Template self-check

Before you ship work from a clone, confirm:

- [ ] `PROJECT.md` front-matter has real `project`, package paths, and **verification gates** (no `my-app` placeholder).
- [ ] `stack_profile` points at the profile you actually use.
- [ ] `target_platforms` matches reality (`[web]`, `[web, ios, android]`, or `[backend-only]`).
- [ ] Your gate commands run locally and exit `0` on a clean checkout.
- [ ] `ARCHITECTURE.md` reflects your real services/data model once the system is non-trivial.
- [ ] `.dev/tasks/` is git-ignored.

---

## Background reading

The design is grounded in a survey of current agentic-development tools and patterns, captured in [`docs/research/`](docs/research/):

- [`01-agentic-frameworks.md`](docs/research/01-agentic-frameworks.md) — LangGraph, CrewAI, AutoGen, MetaGPT, OpenHands, aider, Claude Code, and more.
- [`02-spec-driven-dev.md`](docs/research/02-spec-driven-dev.md) — Kiro, spec-kit, and spec-as-source-of-truth patterns.
- [`03-deep-research-patterns.md`](docs/research/03-deep-research-patterns.md) — structured research and citation discipline.
- [`04-autonomous-dev-test-harness.md`](docs/research/04-autonomous-dev-test-harness.md) — local servers, sandboxes, and self-test loops.
- [`05-cross-platform.md`](docs/research/05-cross-platform.md) — shared-codebase web + iOS + Android rules.

---

## License

[MIT](LICENSE) © Tanav Aggarwal. This template ships no product code and imposes no runtime dependencies of its own.
