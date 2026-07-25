---
name: launch-subagent
description: 'Read this BEFORE launching any subagent (Task tool, background agents, parallel agents, best-of-N, delegating work to another agent). Stack A model policy (orchestrator / junior DeepSeek / senior Grok / Opus sign-off) plus consensus principles. Triggers: launch a subagent, spawn agents, run agents in parallel, delegate to a subagent.'
---

# Stack A — Quality-first model policy

Hard defaults for this skills pack. Override only if the user explicitly names a different model.

| Role | Model | Notes |
|------|--------|--------|
| **Orchestrator** | **Opus 5** (host session) | Plans, briefs, routes work, integrates. Does not grind most implementation. |
| **Junior engineer** (default implementer) | **DeepSeek V4 Pro** | Does **most of the work**. Use `deepseek-subagent`. Full brief + verification commands required. |
| **Senior engineer** | **Grok 4.5** | Reviews junior output and **fixes** serious issues. `grok-review` + `grok-subagent` for fixes. |
| **Final sign-off** | **Opus 5** | Last gate before done/merge. `opus-review` and/or orchestrator verification. |

## Default pipeline (non-trivial implementation)

1. **Opus** — plan + write self-contained brief.
2. **DeepSeek V4 Pro** — implement (`deepseek-subagent`).
3. **Grok 4.5** — review; fix Critical/Important issues (`grok-review`, then `grok-subagent` if fixes needed).
4. **Opus 5** — final sign-off (diff + tests). Report to user.

Do **not** skip senior Grok review on non-trivial DeepSeek work. Do **not** claim done without Opus sign-off.

## Orchestrator rules

- The **host session** is the orchestrator. Prefer starting that session on **Opus 5**.
- The orchestrator does **not** silently re-assign itself to a cheap model mid-task for planning/sign-off.
- Orchestrator **always** inspects executor diffs / test output before claiming done.
- DO NOT launch subagents unless the User tells you to, or the task is clearly large independent work.

## Executor rules

### Most work → DeepSeek V4 Pro (junior)

- Default implementer for features, bugfixes, tests, refactors with a complete brief.
- Auth: `source ~/.config/stack-a/env` — uses `DEEPSEEK_API_KEY` (never print the key).
- Skill: **`deepseek-subagent`**.
- After DeepSeek finishes: run verification; then send to **Grok senior** review.

### Senior review / fix → Grok 4.5

- After junior implementation: **`grok-review`** (report findings).
- If Critical/Important issues: **`grok-subagent`** to fix (or tightly scoped re-brief to DeepSeek for tiny typos only).
- Grok may also take **hard rescue** when DeepSeek is blocked (auth, multi-system design, security-sensitive work).

### Local Ollama / Qwen — NOT default (almost never)

- **Do not** run `ollama list`, pick a local model, or brief work “for local Qwen/Gemma” just because Ollama is installed.
- **Do not** use local models for investigate / debug / implement when the user says “use Stack A” without naming a local model.
- Local Qwen is **opt-in only**: user explicitly asks for local/Ollama **or** DeepSeek is **proven down** (missing key / 401 / API unreachable) **and** the task is tiny/mechanical.
- If DeepSeek is down for a non-trivial task → escalate to **Grok 4.5** (senior), not Ollama.
- Order when (and only when) local is allowed: **Qwen3.6** if installed, else **`qwen3.5:9b`**. Never invent other local picks (gemma, gpt-oss, etc.) unless the user names them.

### Banned as subagent defaults

- Do **not** default implementers to Composer, Sonnet-class, or random auto-picks.
- Do **not** default to **Ollama / local models** when Stack A is requested.
- Do **not** use OpenAI Codex / GPT Sol as the Stack A default (legacy `codex-subagent` only if the user explicitly asks for Codex).

## Reviewer / sign-off rules

| Stage | Who | Skill |
|-------|-----|--------|
| Senior review of junior work | **Grok 4.5** | `grok-review` |
| Senior fixes | **Grok 4.5** | `grok-subagent` |
| Final sign-off | **Opus 5** | `opus-review` / host Opus |

- Reviewers stay **neutral**; show reports **verbatim**.
- Local Qwen / cheap models may run a **checklist self-check** only — never sole merge-gate or final sign-off.

## How subagents actually run

| Launch style | What happens |
|--------------|--------------|
| **DeepSeek junior** | `deepseek-subagent` — OpenAI-compatible API (`DEEPSEEK_*` env). Separate process / agent loop. |
| **Grok senior** | `grok -p` / `--prompt-file` — **new OS process**. |
| **Host Task** | Only if the host can select the right model family for the role. |
| **Opus sign-off** | Host Opus session or Task on Opus 5. |

**Claude Code specifically:** DeepSeek and Grok run as **separate processes** (or API agents), not “inside” Opus. Opus remains orchestrator + final sign-off.

---

# SUBAGENTS (hard rules — repeat)

- Orchestrator host: **Opus 5**
- Junior implementer (most work): **DeepSeek V4 Pro** via `deepseek-subagent`
- Senior engineer: **Grok 4.5** review + fix (`grok-review` / `grok-subagent`)
- Final sign-off: **Opus 5** (`opus-review`)
- DO NOT launch subagents unless the User tells you to (or clear parallel independent work)
- NEVER default subagents to Composer / weak auto models / Ollama
- only ever use Stack A models above unless the user overrides
- Default route when user says “use Stack A and delegate”: **DeepSeek junior** (not Grok first, not Ollama)

## General Subagent Principles

Consensus of Boris Cherny, Matt Pocock, Pietro Schirano, and Peter Steinberger:

- Delegate only self-contained tasks. Split work so subtasks have zero dependencies on each other; parallelize only independent work.
- Parallel subagents must never touch the same files — that is a recipe for conflicts. Partition the work or keep it in one agent.
- Subagents start blind: they see none of your context. Write the full brief into the prompt — scope, all needed context, constraints, and the exact output to return.
- Scope narrowly and concretely: "explore how payments work" beats "explore everything". One bounded task per subagent, small blast radius.
- The main agent stays the orchestrator. It plans the split, integrates results, and reviews/verifies every subagent output before trusting it.
- Keep critical planning, tightly-coupled judgment, and final sign-off in the Opus loop — implementation bulk goes to DeepSeek; hard fixes to Grok.
- Have subagents return short summaries or concrete results, never raw transcripts or file dumps. That keeps the main context clean.

# SUBAGENTS (hard rules — repeat)

- Orchestrator host: **Opus 5**
- Junior implementer (most work): **DeepSeek V4 Pro** via `deepseek-subagent`
- Senior engineer: **Grok 4.5** review + fix (`grok-review` / `grok-subagent`)
- Final sign-off: **Opus 5** (`opus-review`)
- DO NOT launch subagents unless the User tells you to (or clear parallel independent work)
- NEVER default subagents to Composer / weak auto models / Ollama
- only ever use Stack A models above unless the user overrides
- Default route when user says “use Stack A and delegate”: **DeepSeek junior** (not Grok first, not Ollama)
