---
name: launch-subagent
description: 'Read this BEFORE launching any subagent (Task tool, background agents, parallel agents, best-of-N, delegating work to another agent). Stack A model policy (orchestrator / junior DeepSeek / senior Grok / light Sonnet ops / Opus sign-off) plus consensus principles. Triggers: launch a subagent, spawn agents, run agents in parallel, delegate to a subagent.'
---

# Stack A — Quality-first model policy

Hard defaults for this skills pack. Override only if the user explicitly names a different model.

| Role | Model | Notes |
|------|--------|--------|
| **Orchestrator** | **Opus 5** (host session) | Plans, briefs, routes work, integrates. Does not grind most implementation. |
| **Junior engineer** (default implementer) | **DeepSeek V4 Pro** | Does **most of the coding**. Use `deepseek-subagent` (Hermes). Full brief required. |
| **Senior engineer** | **Grok 4.5** | Reviews junior output and **fixes** serious issues. `grok-review` + `grok-subagent`. |
| **Light ops** | **Claude Sonnet 5** | **Commit, merge, push, open PR**, branch cleanup — mechanical ship steps after work is decided. **Not** feature implementation. |
| **Final sign-off** | **Opus 5** | Last gate before done/merge on non-trivial work. `opus-review`. |

## Default pipeline (non-trivial implementation)

1. **Opus** — plan + write self-contained brief.
2. **DeepSeek V4 Pro** — implement (`deepseek-subagent`).
3. **Grok 4.5** — review; fix Critical/Important issues.
4. **Opus 5** — final sign-off (diff + tests).
5. **Sonnet 5** (optional) — commit / merge / push / PR when the user asks for light ship steps.

Do **not** skip senior Grok review on non-trivial DeepSeek work. Do **not** claim done without Opus sign-off on non-trivial work.

## Orchestrator rules

- The **host session** is the orchestrator. Prefer starting that session on **Opus 5**.
- The orchestrator does **not** silently re-assign itself to a cheap model mid-task for planning/sign-off.
- Orchestrator **always** inspects executor diffs / test output before claiming done on code work.
- DO NOT launch subagents unless the User tells you to, or the task is clearly large independent work.
- **Route by task weight:** code judgment → DeepSeek/Grok/Opus; pure git ceremony → Sonnet.

## Executor rules

### Most coding → DeepSeek V4 Pro (junior)

- Default implementer for features, bugfixes, tests, refactors with a complete brief.
- Auth: `source ~/.config/stack-a/env` — `DEEPSEEK_API_KEY` (never print the key).
- Skill: **`deepseek-subagent`** (Hermes + deepseek-v4-pro).
- After DeepSeek finishes: verification; then **Grok senior** review.

### Senior review / fix → Grok 4.5

- After junior implementation: **`grok-review`**.
- Critical/Important issues: **`grok-subagent`**.
- Hard rescue when DeepSeek is blocked (auth, multi-system design, security-sensitive work).

### Light ops → Claude Sonnet 5

Use Sonnet when the task is **mechanical and already decided** — a less capable model is enough and saves expensive tokens.

**In scope**

- Commit with a clear message (orchestrator may draft the message)
- Merge feature branch → main when tests already reported green
- Push / set upstream
- Open PR with provided title/body
- Delete merged branch, simple git hygiene

**Out of scope (do not use Sonnet)**

- Designing or implementing features
- Ambiguous debugging
- Code review judgment
- Force-push / history rewrite / prod without explicit user order
- Anything needing “is this correct?” on non-trivial code

**How to launch:** host Task / agent session with model **Claude Sonnet 5** (or the closest available Sonnet slug in that product). Keep the brief = exact commands + constraints (“no force push”, “run tests first if not already green”).

### Local Ollama / Qwen — NOT default (almost never)

- **Do not** run `ollama list` or pick local models just because Ollama is installed.
- Local Qwen is **opt-in only**: user asks for local **or** DeepSeek is proven down **and** task is tiny/mechanical.
- DeepSeek down + non-trivial code → **Grok 4.5**, not Ollama.
- Order when local is allowed: **Qwen3.6** if installed, else **`qwen3.5:9b`**.

### Banned as subagent defaults

- Do **not** default **feature implementation** to Composer, Sonnet, or random auto-picks. (Sonnet is **light ops only**.)
- Do **not** default to **Ollama** when Stack A coding is requested.
- Do **not** use OpenAI Codex / GPT Sol as Stack A default (only if user asks).

## Reviewer / sign-off rules

| Stage | Who | Skill |
|-------|-----|--------|
| Senior review of junior work | **Grok 4.5** | `grok-review` |
| Senior fixes | **Grok 4.5** | `grok-subagent` |
| Final sign-off (non-trivial) | **Opus 5** | `opus-review` / host Opus |
| Light ship steps | **Sonnet 5** | host Task |

- Reviewers stay **neutral**; show reports **verbatim**.
- Sonnet / local Qwen never sole merge-gate on non-trivial code.

## How subagents actually run

| Launch style | What happens |
|--------------|--------------|
| **DeepSeek junior** | `deepseek-subagent` / Hermes — API + tool loop |
| **Grok senior** | `grok -p` / `--prompt-file` — **new OS process** |
| **Sonnet light ops** | Host Task / Claude session on **Sonnet 5** |
| **Opus sign-off** | Host Opus or Task on Opus 5 |

**Claude Code:** DeepSeek and Grok are separate processes/API agents. Sonnet is a lighter **same-family** host model for git ceremony only. Opus remains orchestrator + final sign-off for real work.

---

# SUBAGENTS (hard rules — repeat)

- Orchestrator host: **Opus 5**
- Junior implementer (most coding): **DeepSeek V4 Pro** via `deepseek-subagent`
- Senior engineer: **Grok 4.5** review + fix
- Light ops (commit / merge / push / PR): **Claude Sonnet 5**
- Final sign-off (non-trivial): **Opus 5**
- DO NOT launch subagents unless the User tells you to (or clear parallel independent work)
- NEVER default **coding** to Sonnet / Composer / Ollama
- Default “use Stack A and delegate” for **code** → **DeepSeek junior**
- Default for **commit/merge/push only** → **Sonnet light ops**

## General Subagent Principles

Consensus of Boris Cherny, Matt Pocock, Pietro Schirano, and Peter Steinberger:

- Delegate only self-contained tasks. Split work so subtasks have zero dependencies on each other; parallelize only independent work.
- Parallel subagents must never touch the same files — that is a recipe for conflicts. Partition the work or keep it in one agent.
- Subagents start blind: they see none of your context. Write the full brief into the prompt — scope, all needed context, constraints, and the exact output to return.
- Scope narrowly and concretely: "explore how payments work" beats "explore everything". One bounded task per subagent, small blast radius.
- The main agent stays the orchestrator. It plans the split, integrates results, and reviews/verifies every subagent output before trusting it.
- Keep critical planning, tightly-coupled judgment, and final sign-off in the Opus loop — implementation bulk to DeepSeek; hard fixes to Grok; git ceremony to Sonnet.
- Have subagents return short summaries or concrete results, never raw transcripts or file dumps. That keeps the main context clean.

# SUBAGENTS (hard rules — repeat)

- Orchestrator host: **Opus 5**
- Junior implementer (most coding): **DeepSeek V4 Pro** via `deepseek-subagent`
- Senior engineer: **Grok 4.5** review + fix
- Light ops (commit / merge / push / PR): **Claude Sonnet 5**
- Final sign-off (non-trivial): **Opus 5**
- DO NOT launch subagents unless the User tells you to (or clear parallel independent work)
- NEVER default **coding** to Sonnet / Composer / Ollama
- Default “use Stack A and delegate” for **code** → **DeepSeek junior**
- Default for **commit/merge/push only** → **Sonnet light ops**
