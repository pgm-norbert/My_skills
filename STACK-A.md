> **Updated 2026-08-07 — lineup changed.**
> Senior engineer / implementer: **Grok 4.5** (`grok-subagent`).
> Junior worker: **GPT-5.6 Luna** (`luna-subagent`).
> Reviewer: **GPT-5.6 Sol**, falling back to **GPT-5.5** (`sol-review`) — the
> reviewer is deliberately a different vendor from the implementer.
> Orchestration and sign-off remain **Opus 5** (`opus-review`).
> DeepSeek is retired (its balance ran out 2026-08-07). `gpt-5.6-sol` **is** callable
> on a ChatGPT-account Codex login — verified 2026-08-07.
> Canonical policy now lives in `~/.claude/CLAUDE.md`.

# Stack A — Quality-first orchestration

Personal policy: **Opus orchestrates and signs off**, **Grok 4.5 does most of the coding**, **GPT-5.6 Sol reviews adversarially** (a different vendor from the implementer, deliberately), **Sonnet handles light ops** (git commit / merge / push when the work is already decided).

## Roles

| Role | Model | Job |
|------|--------|-----|
| **Orchestrator** | **Opus 5** | Host session. Plans, briefs, routes work, integrates. Does **not** grind implementation by default. |
| **Senior engineer** (default implementer) | **Grok 4.5** | Does **most of the coding**: features, fixes, tests from a complete brief. Skill: `grok-subagent`. |
| **Junior worker** | **GPT-5.6 Luna** | Bounded mechanical work: bulk edits, classification, short tasks. Skill: `luna-subagent`. **Not** the default feature implementer. |
| **Reviewer** | **GPT-5.6 Sol** (falls back to GPT-5.5 / Terra) | Adversarial review of the implementer's work. Skill: `sol-review`. Deliberately a **different vendor** from the implementer. |
| **Light ops** | **Claude Sonnet 5** | Cheap mechanical tasks once the plan is clear: **commit, merge, push, open PR, branch hygiene**, rename files, apply a one-line fix the senior already specified. **Not** for designing or implementing features. |
| **Final sign-off** | **Opus 5** | Last gate before “done” / merge claim on non-trivial work. Skill: `opus-review`. |

## Worker lanes: Grok vs Luna

Two lanes below the orchestrator, for different jobs:

| Lane | Slug | Role | Use when |
|------|------|------|----------|
| **Grok 4.5** (default implementer) | `grok-4.5` | Implementer for **coding work** | Features, multi-file fixes, tests, refactors — anything that needs a full brief and tool loop |
| **GPT-5.6 Luna** (junior worker) | `gpt-5.6-luna` | Bounded mechanical work | Bulk edits, classification, short rewrites, labels — anything fully specified up front |

### Rules of thumb

- **“Use Stack A and delegate” (code)** → **Grok 4.5** via `grok-subagent`.
- **Bounded mechanical work / bulk edits / classification** → **Luna** via `luna-subagent`.
- **Do not** default non-trivial multi-file coding to Luna to “save money” — Grok is the implementer; Luna is the mechanical lane.
- User can override: “use Luna for this patch” or “use Grok for this” — honor explicit requests.
- After **Pro** coding: still **Grok senior review** + **Opus sign-off** on non-trivial work. Flash chat output is not a merge gate.

### Product apps

When wiring an LLM into an app (chat FAB, retrieval answer, etc.), prefer:

1. **Flash** for interactive user-facing chat / RAG (speed + cost).
2. **Pro** only if the product task is multi-step coding-style generation that Flash fails on.
3. Never put API keys in the repo — load from env / local secrets.

## Default delivery pipeline (non-trivial implementation)

```
User
 └─ Opus 5 (orchestrator): plan + brief + definition of done
      └─ Grok 4.5 (implementer): implement most of the task
      └─ GPT-5.6 Sol (reviewer): adversarial review; cross-vendor from the implementer
      └─ Opus 5 (sign-off): final check of diff + tests
      └─ Sonnet 5 (light ops, optional): commit / merge / push when user asks
```

1. **Opus** writes a self-contained brief.
2. **Grok 4.5** implements (`grok-subagent`).
3. **Sol** reviews adversarially (`sol-review`); Grok fixes what it finds.
4. **Opus** signs off on non-trivial work.
5. **Sonnet** may run **light ops** (commit/merge/push/PR) so Opus/Grok/Sol tokens aren’t spent on git ceremony.

## When to use Sonnet 5 (light ops only)

**In scope for Sonnet**

- `git status` / `git diff` summary the user already approved
- Commit with a message the orchestrator drafted (or conventional-commit from a clear diff)
- Merge feature branch → main (or restack) when tests already green
- Push to origin / set upstream
- Open a PR with title/body provided or templated from the commit
- Trivial follow-through: delete merged branch, tag after release notes are written

**Out of scope for Sonnet — escalate**

| Task | Route instead |
|------|----------------|
| Feature / bugfix / refactor | **Grok 4.5** implementer |
| Ambiguous “fix ranking” / design | Opus plan → Grok 4.5 |
| Bulk edit / classification / label | **GPT-5.6 Luna** |
| Code review judgment | Grok senior |
| Final “is this safe to ship?” on non-trivial work | Opus sign-off |
| Force-push, rewrite history, prod deploys without clear user order | Opus (or ask user) |

**Rule:** if the task needs **judgment about code correctness**, do **not** use Sonnet. If the task is **executing an already-decided git/ship step**, Sonnet is preferred.

## Credentials

- **Never** put API keys in skill markdown or git.
- Local file: `~/.config/stack-a/env` (mode `600`), sourced from `~/.zshrc`.
- **Grok 4.5 (implementer):** the `grok` CLI, authenticated on its own. No key needed in this file.
- **GPT-5.6 Sol / Luna (reviewer / junior):** the `codex` CLI. A ChatGPT login is sufficient —
  `gpt-5.6-sol` is callable that way (verified 2026-08-07).

```bash
which grok  && grok --version    # implementer lane
which codex && codex login status # reviewer / junior lane
```

## Local Ollama — not the default path

| User says | Route |
|-----------|--------|
| “use Stack A and delegate …” (code) | **Grok 4.5** → Sol review → Opus |
| “bulk edit / classify / label” | **GPT-5.6 Luna** |
| “commit / merge / push” (light) | **Sonnet 5** light ops |
| “use local / Ollama …” | Only then local Qwen |
| Grok down + non-trivial code | **GPT-5.6 Sol** (implement, then Opus signs off) |

Do **not** open with `ollama list` when Stack A was requested for coding.

## Banned as defaults

- **Ollama / local models** as the first pick for coding under Stack A.
- **Sonnet** as the **default implementer** for features (only light ops).
- **Flash** as the default **feature** implementer (Flash = chat/fast; Pro = coding junior).
- **Codex / GPT Sol** — only if the user explicitly asks.
- Skipping **Sol** adversarial review on non-trivial Grok output.
- Claiming merge-ready without **Opus** sign-off after a real implementation pipeline.
- Briefing **Grok first** for bulk implementation (Grok is senior, not default junior).

## Skills map

| Skill | Model / role |
|-------|----------------|
| `launch-subagent` | Policy (this stack) |
| `grok-subagent` | Default implementer (**Grok 4.5**) |
| `luna-subagent` | Junior worker (**GPT-5.6 Luna**) — bounded mechanical work |
| `sol-review` | Adversarial review (**GPT-5.6 Sol** → GPT-5.5) |
| `grok-subagent` | Senior fix / hard rescue (Grok 4.5) |
| `grok-review` | Senior review (Grok 4.5) |
| `opus-review` | Final sign-off (Opus 5) |
| Host Task / Claude **Sonnet 5** | Light ops: commit, merge, push, PR |
| Product chat / RAG clients | Pick per app; Stack A does not mandate one |
| `fable-review` | Alias → `opus-review` |
| `gpt-review` | Alias → `grok-review` |
| `codex-subagent` | Legacy — user must ask |

## Install / sync

Skills live under `skills/agent-orchestration/`. Symlink or copy into agent skill roots (`~/.agents/skills`, `~/.claude/skills`, etc.).
