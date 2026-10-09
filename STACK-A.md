> **Updated 2026-08-15.**
> Implementer: **the newest Grok (4.7)**. Reviewer: **newest GPT flagship (GPT-6-Astra) → older flagships** (`sol-review`).
> Junior mechanical: **newest GPT Luna**. Orchestrator depends on the host (below).
> DeepSeek is retired (balance ran out 2026-08-07).
> Canonical live policy: `~/.claude/CLAUDE.md`. Grok-host skill: `stack-a-grok-host`.

# Stack A — Quality-first orchestration

Two principles in every mode:
- **Cross-vendor review** — the reviewer is never the same vendor as the implementer.
- **Verify, don't trust** — the orchestrator re-runs tests / reads the diff / looks at the app before "done".


## Model freshness rule (always the newest)

Every seat uses the **newest model its vendor offers on this login** — never a pinned
version. Before launching, discover them (no network, no tokens):

```bash
eval "$(~/.config/stack-a/stack-a-models)"
# GROK_IMPLEMENTER  newest grok-X.Y        (from `grok models`)
# GPT_REVIEWERS     newest GPT flagship first, older flagships as fallback
# GPT_JUNIOR        newest luna/mini model
```

Resolved 2026-10-09: implementer **grok-4.7**; reviewer **gpt-6-astra** → gpt-5.6-sol →
gpt-5.6-terra → gpt-5.5; junior **gpt-5.6-luna**. When a vendor ships a newer model the
script picks it up automatically; only the "Resolved" line in docs goes stale.
If the ChatGPT account hits its usage limit, every GPT is out until the reset time —
wait, or (with the user's OK) review with a non-orchestrator Claude model (`fable-review`).

## Host modes

Only the orchestrator seat moves. Grok always implements. A GPT always reviews.

| Mode | When | Orchestrator | Implementer | Reviewer | Sign-off |
|------|------|--------------|-------------|----------|----------|
| **Normal** | Claude Code, Opus budget OK | Opus 5 | Grok (`grok-subagent`) | newest GPT (Astra) → fallbacks | Opus |
| **Opus-Saver** | Opus session usage ≈ 70%+ | Sonnet 5 (`/model sonnet`) | Grok (`grok-subagent`) | newest GPT (Astra) → fallbacks | Sonnet; Opus only for high-risk |
| **Grok-host** | Opus at session limit, or user opened Grok TUI | **This Grok session** | **This session** (do not spawn `grok-subagent`) | newest GPT (Astra) → fallbacks | This session; defer high-risk Opus |

Grok-host procedure: skill **`stack-a-grok-host`**. Always-on Grok rule: `~/.grok/rules/stack-a-grok-host.md`.

## Roles

| Role | Model | Job |
|------|--------|-----|
| **Orchestrator** | Opus / Sonnet / Grok-host | Plans, briefs, routes, integrates, talks to the user. |
| **Senior engineer** | **the newest Grok (4.7)** | Most coding. Skill: `grok-subagent` from a non-Grok host; in-session on Grok-host. |
| **Junior worker** | **newest GPT Luna** | Bounded mechanical work. Skill: `luna-subagent`. Not the default implementer. |
| **Reviewer** | **Newest GPT flagship** (GPT-6-Astra) → older flagships | Adversarial review. Skill: `sol-review`. Different vendor from the implementer. |
| **Light ops** | **Claude Sonnet 5** | Commit / merge / push / PR when the work is already decided. On Grok-host, the host may do this. |
| **Final sign-off** | **Opus 5** (Normal) | Last gate on non-trivial work. Skill: `opus-review`. Saver/Grok-host: the host verifies; high-risk waits for Opus. |

## Default delivery pipeline

```
User
 └─ Host orchestrator (Opus / Sonnet / Grok)
      └─ Grok implements (subagent from Claude; in-session from Grok TUI)
      └─ newest GPT (Astra) → fallbacks reviews
      └─ Grok fixes
      └─ Host verifies independently
```

1. Host writes a self-contained brief (or implements, if Grok-host).
2. Grok implements.
3. the newest GPT reviews (`sol-review`); Grok fixes.
4. Host signs off with evidence.
5. Sonnet (or the Grok host) may run git ceremony when the user asks.

## When to use Sonnet 5 (light ops only)

**In scope:** `git status` / `git diff` the user already approved; commit; merge; push; open a PR; delete a merged branch.

**Out of scope — escalate**

| Task | Route |
|------|--------|
| Feature / bugfix / refactor | Grok (subagent or Grok-host) |
| Bulk edit / classification | Luna |
| Code review judgment | `sol-review` |
| Final "safe to ship?" on high-risk work | Opus, when available |

## Credentials

- **Never** put API keys in skill markdown or git.
- Local file: `~/.config/stack-a/env` (mode `600`).
- **Grok:** the `grok` CLI, authenticated on its own.
- **Sol / Luna / Terra:** the `codex` CLI. ChatGPT login is sufficient.
  Re-probe Sol each session — it has flipped available/unavailable before.

```bash
which grok  && grok --version
which codex && codex login status
```

## Local Ollama — not the default path

| User says | Route |
|-----------|--------|
| implement / build / fix / refactor | Grok → GPT review → host verify |
| bulk edit / classify | Luna |
| commit / merge / push | Sonnet light ops, or Grok-host |
| use local / Ollama | Only then local Qwen |

Do **not** open with `ollama list` when Stack A was requested for coding.

## Banned as defaults

- Ollama / local models as the first pick for coding.
- Sonnet or Luna as the default feature implementer.
- Grok reviewing Grok.
- Skipping GPT review on non-trivial Grok output.
- Claiming merge-ready on an agent's report alone.
- Following `deepseek-subagent` for new work.
- From Grok TUI: launching `grok-subagent` instead of implementing here.

## Skills map

| Skill | Model / role |
|-------|----------------|
| `launch-subagent` | Routing policy (read first) |
| `stack-a-grok-host` | Grok TUI is the host — implement here |
| `grok-subagent` | Default implementer from a non-Grok host |
| `luna-subagent` | Junior worker — bounded mechanical work |
| `sol-review` | Adversarial review (newest GPT (Astra) → fallbacks) |
| `opus-review` | Final sign-off (Opus host) |
| `grok-review` | Stub → `sol-review` |
| `fable-review` | Alias → `opus-review` |
| `gpt-review` | Alias → `sol-review` |
| `deepseek-subagent` | **DEPRECATED** — do not route new work |
| `codex-subagent` | Legacy — user must ask |

## Install / sync

Skills live under `skills/agent-orchestration/`. Symlink into agent skill roots
(`~/.agents/skills`, `~/.claude/skills`, `~/.grok/skills`). Grok-host also has
an always-on rule at `~/.grok/rules/stack-a-grok-host.md`.
