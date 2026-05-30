# Agentic software-building frameworks: dense research notes

Scope: reusable ideas for a generic, publishable `idea -> spec -> plan -> build -> verify -> ship` multi-agent template. Favor patterns that are tool/model-agnostic, file-based, auditable, and easy to clone.

## Compact comparison

| Tool | Type | Workflow loop | Review/test approach | Reusable idea |
|---|---|---|---|---|
| [LangGraph](https://langchain-ai.github.io/langgraph/) | Stateful orchestration graph | Nodes are agents/tools; edges define sequential, conditional, cyclic flows; state/checkpoints persist between steps. | Tests can target nodes/graphs; human approval nodes and validators are natural gates. | Represent the lifecycle as an explicit state machine, not just a prompt; persist state for resume/replay. |
| [CrewAI](https://docs.crewai.com/concepts/crews) | Role/task multi-agent framework | `Crew` = agents + tasks + process; sequential passes task output as context; hierarchical uses manager agent/LLM for planning, delegation, validation. | Step/task callbacks, tracing, checkpointing; manager reviews task completion in hierarchical process ([process docs](https://docs.crewai.com/concepts/processes)). | Separate `agents.yaml` from `tasks.yaml`; support sequential default plus manager mode for complex work. |
| [Microsoft AutoGen / AgentChat](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html) | Multi-agent app framework | Agents are grouped into Teams; patterns include SelectorGroupChat, Swarm, Magentic-One, GraphFlow. | GraphFlow gives deterministic sequential/parallel/conditional/looped agent execution; logging/memory built in ([GraphFlow](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/graph-flow.html)). | Offer both conversational teams and strict DAG flows; use directed graphs when phase ordering matters. |
| [OpenAI Swarm](https://github.com/openai/swarm) | Lightweight educational handoff framework | `client.run`: completion -> tool calls -> optional agent switch -> context update -> repeat until no tool calls. | Designed to be lightweight/testable; no server-side state. | Handoff primitive: an agent is just instructions + tools; returning another agent transfers control. |
| [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) | Production agent runtime | Built-in agent loop manages tools, handoffs, sessions, guardrails, sandbox agents, tracing. | Guardrails run validation in parallel/fail fast; tracing supports debugging/evals. | Keep primitives few: Agent, tool, handoff, guardrail, session, trace. Add sandboxed specialists for repo work. |
| [MetaGPT](https://github.com/geekan/MetaGPT) | Software-company multi-agent framework | One-line requirement -> PM/architect/PM/engineer SOP pipeline -> user stories, requirements, APIs, data structures, docs, code. | SOP roles include review/test-style responsibilities; outputs multiple planning artifacts before code. | Encode proven software SOPs as reusable role prompts and artifact contracts: `Code = SOP(Team)`. |
| [ChatDev](https://github.com/OpenBMB/ChatDev) | Research software-company simulation | Requirements/design/coding/testing roles communicate in chat to build small software. | Explicit reviewer/tester personas in the simulated company loop. | Use role diversity for blind critique, but keep it bounded by artifacts to avoid chat sprawl. |
| [OpenHands](https://docs.openhands.dev/) | Autonomous software engineering platform/SDK | Agent reasons/actions against workspace through tools; CLI/GUI/cloud share SDK. | SDK includes sandbox isolation, persistence, observability, security confirmation, critics/iterative refinement in docs index ([llms.txt](https://docs.openhands.dev/llms.txt)). | Event/action log + isolated workspace + resumable conversations = auditable autonomous coding. |
| [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) | Autonomous workflow/agent platform | Modern platform uses low-code block workflows; classic loop plans/actions/memory; agents can run continuously. | Platform emphasizes testing, deployment controls, monitoring, benchmark (`agbenchmark`) for agents. | For generic framework: prefer explicit blocks/tasks over unconstrained goal loops; benchmark agent behavior. |
| [BabyAGI](https://github.com/yoheinakajima/babyagi) | Minimal autonomous task-loop pattern | Objective -> create task -> execute -> store result -> create/prioritize next tasks -> repeat. | Early pattern had weak verification; relies on stopping criteria and memory quality. | Useful as a caution: task queues need hard completion criteria, budgets, and verification gates. |
| [Devin](https://www.cognition.ai/blog/introducing-devin) | Commercial autonomous software engineer | Takes engineering task, plans, codes, uses shell/browser/editor, reports progress. | Marketed around validating with tests and fixing failures; strong UI/progress feedback. | Long-running autonomous tasks need visible plan/progress/checkpoints and human takeover. |
| [aider](https://aider.chat/docs/usage.html) | Git-native coding CLI | User selects/mentions files; aider edits via diff, auto-commits, repo-map adds context. | Auto-lint edited files; `/test` or `--auto-test` runs tests and asks model to fix failures ([lint/test](https://aider.chat/docs/usage/lint-test.html)). | Git as transaction log; tight edit-test-fix loop; add only needed files to context. |
| [GPT-Engineer](https://github.com/AntonOsika/gpt-engineer) | Prompt-to-code generator/agent lab | Natural-language `prompt` file -> generate/execute code; `-i` improves existing projects. | Includes benchmark interface for custom agents against APPS/MBPP. | Keep the initial brief as a file; make agent experiments benchmarkable. |
| [SWE-agent](https://github.com/SWE-agent/SWE-agent) / [mini-SWE-agent](https://github.com/SWE-agent/mini-swe-agent) | Research coding agent for GitHub issues/SWE-bench | LM operates through an agent-computer interface to inspect/edit/run commands until patch. | Evaluated on SWE-bench; configurable via YAML; CI badges/pre-commit in project. | Small, hackable loop + strong environment interface beats elaborate personas for bug-fixing. |
| [Cline](https://docs.cline.bot/cline-overview.md) | IDE/CLI coding agent | Plan & Act modes; reads files, edits code, runs commands with approval; checkpoints. | Checkpoints rollback; Plan mode before edits; supports PR review/GitHub workflows and subagents in docs index ([llms.txt](https://docs.cline.bot/llms.txt)). | Two-mode UX: planning is non-mutating; acting is permissioned and checkpointed. |
| [Roo Code](https://docs.roocode.com/features/custom-modes) | VS Code agent with modes | Modes specialize role, tools, permissions, when-to-use; Orchestrator can switch/delegate. | Safety by restricting tool/file access per mode; share modes via YAML. | Publish role/mode definitions as portable files with tool permissions and invocation hints. |
| [Cursor](https://docs.cursor.com/) | Agentic IDE | Agent mode edits/runs commands; rules provide persistent project guidance; supports MCP/skills/CLI per docs metadata. | Users review diffs and run tests; rules constrain outputs. | Project-local rules are a lightweight on-ramp; make framework conventions editor-agnostic. |
| [Amazon Kiro](https://kiro.dev/docs/specs/) | Spec-driven agentic IDE | Feature spec -> `requirements.md` -> `design.md` -> `tasks.md`; quick-plan can generate all at once. | Requirements analysis and tasks/dependency tracking; hooks automate checks and artifact sync. | Specs are source of truth; require requirements/design/tasks files before implementation. |
| [Replit Agent](https://docs.replit.com/replitai/agent) | Hosted idea-to-app builder | Plain-language idea -> Plan mode task list -> approved build -> preview/publish. | Tests own work regularly; checkpoints permit rollback; task system supports background review/apply ([task system](https://docs.replit.com/core-concepts/agent/task-system.md)). | Make plan approval, preview, checkpoints, rollback, and publish path first-class. |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) | Agentic coding CLI/IDE/web | Explore codebase, edit files, run commands; supports subagents, skills, hooks, checkpoints, PR review, web/desktop/cloud surfaces. | Built-in code review; hooks can block/validate tool calls; skills `/run` and `/verify` launch/confirm apps ([skills](https://docs.anthropic.com/en/docs/claude-code/skills), [hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)). | File-based extension points (`.claude/skills`, `.claude/agents`, settings hooks) are the model for publishable templates. |

## Cross-cutting patterns

### 1. Orchestration shape
- **Graph/state-machine** (LangGraph, AutoGen GraphFlow): best for phase-gated frameworks because allowed transitions are explicit and resumable.
- **Manager/delegation** (CrewAI hierarchical, Agents SDK handoffs, Roo Orchestrator): best when tasks are numerous or unknown upfront.
- **Task queue loop** (BabyAGI, AutoGPT classic): simple but risky without budgets, validators, and stop conditions.
- **SOP pipeline** (MetaGPT/ChatDev): maps well to software lifecycle but can overproduce artifacts unless phases are lean.

### 2. Planning vs acting gates
- Strong tools distinguish **non-mutating planning/research** from **mutating implementation**: Cline Plan/Act, Kiro requirements/design/tasks, Claude subagents with tool restrictions.
- A publishable template should enforce: `intake -> spec -> plan -> plan review -> implementation -> verification -> audit/ship`.
- Put gates in files + permissions, not just prompts: read-only reviewer agents, implementation agents with edit tools, hooks that block unsafe commands.

### 3. Verification loops
- Best coding agents run real commands and feed failures back: aider auto-lint/auto-test, SWE-agent/SWE-bench, OpenHands critics, Replit self-tests, Claude `/verify`.
- Verification should be declared per task: required tests/build/lint/e2e/manual smoke, expected evidence, who ran it, result.
- Add a reviewer persona only after commands pass or fail with useful evidence; LLM review without tests is weak.

### 4. Context management
- File-based context is common: prompt files (GPT-Engineer), rules (Cline/Cursor), skills/subagents (Claude), YAML agents/tasks (CrewAI), specs (Kiro/spec-kit).
- Use progressive disclosure: top-level workflow file is tiny; load detailed skill/spec only when phase requires it.
- Subagents preserve main context by returning summaries rather than raw logs (Claude, Cline, OpenHands). Use for research, broad code search, and independent review.

### 5. Persistence and auditability
- Git commits/checkpoints are dominant safety rails: aider auto-commits, Replit checkpoints, Claude checkpoints, OpenHands event logs.
- Store task state in repo-local ignored files for resume; store publishable templates in committed `.github/` or `.agent/` directories.
- Every transition should leave an artifact: spec, plan, task list, diff, test log, review decision, release note.

## Lessons for our framework

1. **Make the lifecycle a declared graph.** Ship a small state machine (`research -> prd/spec -> plan -> implement -> review -> audit -> ship`) with legal transitions, required artifacts, and resume behavior.
2. **Use file-defined agents/skills.** Define each role as markdown/YAML with: purpose, when-to-use, allowed tools, inputs, outputs, stop criteria. This is portable across Claude/Cursor/Cline/Roo/GitHub-style agents.
3. **Separate artifact contracts from agent personalities.** Specs, plans, task lists, verification reports, and reviews should be stable markdown schemas; agents are replaceable executors.
4. **Gates must be physical.** Read-only reviewers should not edit. Implementers should not self-approve. Hooks/CI should block missing tests, unreviewed plans, or dangerous commands.
5. **Keep planning dense.** Require enough design for implementation and test strategy, but avoid MetaGPT-style artifact sprawl for small tasks. Complexity classification should scale depth.
6. **Mandate executable verification.** A task is not done until declared commands pass or failures are documented with reason and owner. Reviewer prompts should consume command evidence.
7. **Provide checkpoints/rollback.** Encourage small commits, auto-generated checkpoints, and an undo path before each mutating phase.
8. **Support subagent research/review.** Use isolated-context subagents for broad discovery and blind review; return only findings, citations, and actionable diffs.
9. **Add budget and stop conditions.** Every loop needs max turns/time/cost, clear completion criteria, and escalation/fallback when stuck.
10. **Be editor/runtime agnostic.** Publish the framework as plain files plus optional adapters for Claude Code, Cursor, Cline/Roo, GitHub Copilot, OpenAI Agents SDK, and local scripts.
