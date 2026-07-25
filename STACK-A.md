# Stack A — Quality-first orchestration

Personal policy on top of the davidondrej-style agent skills in this repo.

## Roles

| Role | Model | Skill / mechanism |
|------|--------|-------------------|
| Orchestrator | **Opus 5** | Host session (Claude Code / Cursor on Opus) |
| Hard executor | **Grok 4.5** | `grok-subagent` → separate Grok CLI process |
| Routine executor | **DeepSeek V4** or **Qwen3.6 local** | Only with a complete brief + tests |
| Reviewer | **Opus 5 XOR Grok 4.5** | Opposite of implementer: `opus-review` / `grok-review` |

## Claude Code + Grok: what actually runs?

```
┌─────────────────────────────┐
│  Claude Code (Opus 5)       │  ← orchestrator (this chat)
│  plans, briefs, verifies    │
└─────────────┬───────────────┘
              │ shell / background shell
              ▼
┌─────────────────────────────┐
│  grok -p / --prompt-file    │  ← NEW OS process (Grok CLI)
│  -m grok-4.5 --cwd repo     │     own context, tools, files
└─────────────────────────────┘
```

- Grok does **not** run “inside” Claude’s model call.
- It is a **new CLI process** (foreground wait, or **background** job you poll).
- Optional: open an interactive `grok` TUI in another terminal / cmux pane to watch it live.

## Example flow

User (in Claude Code on Opus 5):

> Implement dark mode for Settings, then review.

1. **Orchestrator (Opus)** writes a brief: files, constraints, `npm test` / typecheck.
2. Opus runs **`grok-subagent`** (headless Grok 4.5) with that brief.
3. Grok process edits the repo and exits with a summary.
4. Opus reads diff + runs tests.
5. Opus runs **`opus-review`** only if a second Opus pass is needed for judgment **or** — if Opus itself implemented — runs **`grok-review`**. After Grok implemented, prefer a clean Opus review Task / fresh session (`opus-review`).
6. Opus reports merge readiness to the user.

## Install / sync

Skills live under `skills/agent-orchestration/`. Symlink or copy into your agent skill roots (`~/.agents/skills`, `~/.claude/skills`, etc.).
