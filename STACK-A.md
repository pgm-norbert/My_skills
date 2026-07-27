# Stack A — Quality-first orchestration

Personal policy: **Opus orchestrates and signs off**, **DeepSeek does most of the coding**, **Grok is the senior engineer**, **Sonnet handles light ops** (git commit / merge / push when the work is already decided).

## Roles

| Role | Model | Job |
|------|--------|-----|
| **Orchestrator** | **Opus 5** | Host session. Plans, briefs, routes work, integrates. Does **not** grind implementation by default. |
| **Junior engineer** (default implementer) | **DeepSeek V4 Pro** | Does **most of the coding**: features, fixes, tests from a complete brief. Skill: `deepseek-subagent` (Hermes). |
| **Fast junior / chat** | **DeepSeek V4 Flash** | Cheap + low-latency: product chat, RAG Q&A, short mechanical edits, classification. **Not** the default feature implementer. |
| **Senior engineer** | **Grok 4.5** | Reviews junior work, **fixes** serious issues, hard rescue. Skills: `grok-review`, `grok-subagent`. |
| **Light ops** | **Claude Sonnet 5** | Cheap mechanical tasks once the plan is clear: **commit, merge, push, open PR, branch hygiene**, rename files, apply a one-line fix the senior already specified. **Not** for designing or implementing features. |
| **Final sign-off** | **Opus 5** | Last gate before “done” / merge claim on non-trivial work. Skill: `opus-review`. |

## DeepSeek lanes: Pro vs Flash

DeepSeek has two models. Stack A uses **both**, for different jobs:

| Lane | Slug | Role | Use when |
|------|------|------|----------|
| **Pro** (default junior) | `deepseek-v4-pro` | Hermes implementer for **coding work** | Features, multi-file fixes, tests, refactors — anything that needs a full brief and tool loop |
| **Flash** (fast junior / chat) | `deepseek-v4-flash` | Fast / cheap inference | In-app chat & RAG answers, short rewrites, labels, “is this on-topic?”, latency-sensitive loops |

### Rules of thumb

- **“Use Stack A and delegate” (code)** → **Pro** via `deepseek-subagent`.
- **Chat / RAG / high-volume cheap calls** (including product features like library chat) → **Flash**.
- **Do not** default non-trivial multi-file coding to Flash to “save money” — Pro is the coding junior; Flash is the fast lane.
- User can override: “use Flash for this patch” or “use Pro for this chat” — honor explicit requests.
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
      └─ DeepSeek V4 Pro (junior): implement most of the task
      └─ Grok 4.5 (senior): review; fix Critical/Important issues
      └─ Opus 5 (sign-off): final check of diff + tests
      └─ Sonnet 5 (light ops, optional): commit / merge / push when user asks
```

1. **Opus** writes a self-contained brief.
2. **DeepSeek Pro** implements (`deepseek-subagent` / Hermes).
3. **Grok** reviews; fixes if needed.
4. **Opus** signs off on non-trivial work.
5. **Sonnet** may run **light ops** (commit/merge/push/PR) so Opus/Grok/DeepSeek tokens aren’t spent on git ceremony.

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
| Feature / bugfix / refactor | DeepSeek **Pro** junior |
| Ambiguous “fix ranking” / design | Opus plan → DeepSeek Pro |
| Fast chat / RAG / label | DeepSeek **Flash** |
| Code review judgment | Grok senior |
| Final “is this safe to ship?” on non-trivial work | Opus sign-off |
| Force-push, rewrite history, prod deploys without clear user order | Opus (or ask user) |

**Rule:** if the task needs **judgment about code correctness**, do **not** use Sonnet. If the task is **executing an already-decided git/ship step**, Sonnet is preferred.

## Credentials (DeepSeek)

- **Never** put API keys in skill markdown or git.
- Local file: `~/.config/stack-a/env` (mode `600`), sourced from `~/.zshrc`.
- Variables:
  - `DEEPSEEK_API_KEY`
  - `DEEPSEEK_BASE_URL` (default `https://api.deepseek.com`)
  - `DEEPSEEK_MODEL` (default **`deepseek-v4-pro`** — coding junior)
  - Optional chat override: `DEEPSEEK_FLASH_MODEL=deepseek-v4-flash` (product apps / chat clients)
- **Coding harness:** **Hermes** (`hermes chat --provider deepseek -m deepseek-v4-pro`). Key also in `~/.hermes/.env`.

```bash
source ~/.config/stack-a/env
test -n "$DEEPSEEK_API_KEY" && echo "DeepSeek ready (coding=$DEEPSEEK_MODEL)"
```

## Local Ollama — not the default path

| User says | Route |
|-----------|--------|
| “use Stack A and delegate …” (code) | **DeepSeek V4 Pro** → Grok → Opus |
| “chat / RAG / fast cheap” | **DeepSeek V4 Flash** |
| “commit / merge / push” (light) | **Sonnet 5** light ops |
| “use local / Ollama …” | Only then local Qwen |
| DeepSeek down + non-trivial code | **Grok 4.5** |

Do **not** open with `ollama list` when Stack A was requested for coding.

## Banned as defaults

- **Ollama / local models** as the first pick for coding under Stack A.
- **Sonnet** as the **default implementer** for features (only light ops).
- **Flash** as the default **feature** implementer (Flash = chat/fast; Pro = coding junior).
- **Codex / GPT Sol** — only if the user explicitly asks.
- Skipping Grok senior review on non-trivial DeepSeek **Pro** output.
- Claiming merge-ready without **Opus** sign-off after a real implementation pipeline.
- Briefing **Grok first** for bulk implementation (Grok is senior, not default junior).

## Skills map

| Skill | Model / role |
|-------|----------------|
| `launch-subagent` | Policy (this stack) |
| `deepseek-subagent` | Junior implementer (**DeepSeek V4 Pro**; Flash optional for cheap/short) |
| `grok-subagent` | Senior fix / hard rescue (Grok 4.5) |
| `grok-review` | Senior review (Grok 4.5) |
| `opus-review` | Final sign-off (Opus 5) |
| Host Task / Claude **Sonnet 5** | Light ops: commit, merge, push, PR |
| Product chat / RAG clients | Prefer **DeepSeek V4 Flash** |
| `fable-review` | Alias → `opus-review` |
| `gpt-review` | Alias → `grok-review` |
| `codex-subagent` | Legacy — user must ask |

## Install / sync

Skills live under `skills/agent-orchestration/`. Symlink or copy into agent skill roots (`~/.agents/skills`, `~/.claude/skills`, etc.).
