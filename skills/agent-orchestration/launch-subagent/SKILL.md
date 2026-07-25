---
name: launch-subagent
description: 'Read this BEFORE launching any subagent (Task tool, background agents, parallel agents, best-of-N, delegating work to another agent). Stack A model policy (orchestrator / executor / reviewer) plus consensus principles. Triggers: launch a subagent, spawn agents, run agents in parallel, delegate to a subagent.'
---

# Stack A — Quality-first model policy

Hard defaults for this skills pack. Override only if the user explicitly names a different model.

| Role | Model | Notes |
|------|--------|--------|
| **Orchestrator** | **Opus 5** (host session) | The main agent you are talking to. Plans, splits work, integrates results, verifies before trust. |
| **Hard executor** | **Grok 4.5** | Non-trivial implementation: multi-file features, hard bugs, refactors. Prefer `grok-subagent` (separate Grok CLI process). |
| **Routine executor** | **DeepSeek V4** or **Qwen3.6 local** | Boilerplate, renames, tests-from-spec, mechanical edits — only with a complete brief + verification commands. |
| **Reviewer** | **Opus 5 XOR Grok 4.5** | Must be a **different family** than the implementer. If Grok built it → Opus reviews. If Opus built it → Grok reviews. |

## Orchestrator rules

- The **host session** is the orchestrator. Prefer starting that session on **Opus 5**.
- The orchestrator does **not** silently re-assign itself to a cheap model mid-task.
- Orchestrator **always** reviews executor diffs / test output before declaring done.
- DO NOT launch subagents unless the User tells you to, or the task is large enough that parallel independent work clearly wins.

## Executor rules

### Hard work → Grok 4.5

- Use **Grok 4.5** (`grok-4.5`) via the **`grok-subagent`** skill (headless Grok CLI).
- Full self-contained brief: goal, constraints, files, definition of done, how to verify.
- After Grok finishes: read the summary, `git status` / diff, run verification yourself.

### Routine work → DeepSeek V4 or Qwen3.6 local

- Only when the orchestrator already wrote a tight spec (files, invariants, acceptance tests).
- Prefer a local/cheap coding agent or API the user has configured; if none is available, keep the work on Grok 4.5 rather than guessing.
- Never use a routine model for security, auth, payments, data-loss risk, or ambiguous product work.

### Banned as subagent defaults

- Do **not** default subagents to Composer, Sonnet-class, or random auto-picks.
- Do **not** use OpenAI Codex / GPT Sol as the Stack A default (legacy `codex-subagent` only if the user explicitly asks for Codex).

## Reviewer rules

- Use **`opus-review`** when the implementer was Grok (or another non-Opus model).
- Use **`grok-review`** when the implementer was Opus (or you need a second-family review after Opus edits).
- Reviewer must stay **neutral** — no nudging toward a preferred solution.
- Show the reviewer's report **verbatim**. Do not rewrite it.
- Cheap models (DeepSeek / Qwen) may run a **checklist self-check**, never the sole merge-gate review on high-stakes code.

## How subagents actually run (Claude Code / Cursor / host agents)

Subagents are **not** “another model inside the same chat bubble” unless the host product has native multi-model Task routing.

| Launch style | What happens |
|--------------|--------------|
| **Grok CLI headless** (`grok -p` / `--prompt-file`) | **New OS process**. Separate context. Can be **backgrounded** from the host’s shell tool. Preferred Stack A hard executor. |
| **Grok interactive TUI** (`grok "…"` in a terminal / cmux pane) | **New terminal session** you can watch. Same model, interactive. |
| **Host Task / Agent tool** (Claude Code Task, Cursor subagent) | Host product spawns a subagent with the **model you select**. If the host cannot select Grok, use Grok CLI instead. |
| **Cheap local / DeepSeek** | Separate process or API call — only if configured on the machine. |

**Claude Code specifically:** asking for Grok work means Claude runs a **shell command** that starts the **Grok CLI**. That is a **new CLI process** (foreground or background), not Grok “inside” Claude. Claude remains orchestrator; Grok is the worker.

---

# SUBAGENTS (hard rules — repeat)

- Orchestrator host: **Opus 5**
- Hard executor: **Grok 4.5** via `grok-subagent` when possible
- Routine executor: **DeepSeek V4** or **Qwen3.6 local** only with a complete brief
- Reviewer: **opposite family** of the implementer (`opus-review` vs `grok-review`)
- DO NOT launch subagents unless the User tells you to (or clear parallel independent work)
- NEVER default subagents to Composer / weak auto models
- only ever use Stack A models above unless the user overrides

## General Subagent Principles

Consensus of Boris Cherny, Matt Pocock, Pietro Schirano, and Peter Steinberger:

- Delegate only self-contained tasks. Split work so subtasks have zero dependencies on each other; parallelize only independent work.
- Parallel subagents must never touch the same files — that is a recipe for conflicts. Partition the work or keep it in one agent.
- Subagents start blind: they see none of your context. Write the full brief into the prompt — scope, all needed context, constraints, and the exact output to return.
- Scope narrowly and concretely: "explore how payments work" beats "explore everything". One bounded task per subagent, small blast radius.
- The main agent stays the orchestrator. It plans the split, integrates results, and reviews/verifies every subagent output before trusting it.
- Keep critical implementation, tightly-coupled edits, and quick fixes in the main loop — delegation overhead is only worth it for independent, research-heavy, or review work.
- Have subagents return short summaries or concrete results, never raw transcripts or file dumps. That keeps the main context clean.

# SUBAGENTS (hard rules — repeat)

- Orchestrator host: **Opus 5**
- Hard executor: **Grok 4.5** via `grok-subagent` when possible
- Routine executor: **DeepSeek V4** or **Qwen3.6 local** only with a complete brief
- Reviewer: **opposite family** of the implementer (`opus-review` vs `grok-review`)
- DO NOT launch subagents unless the User tells you to (or clear parallel independent work)
- NEVER default subagents to Composer / weak auto models
- only ever use Stack A models above unless the user overrides
