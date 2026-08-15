<!-- Generated from ~/.claude/CLAUDE.md — the canonical Stack A policy.
     Paste this block into ~/.claude/CLAUDE.md on a new machine. -->

# Stack A — quality-first orchestration (always on)

You are the **orchestrator** (this host session). Two principles in every mode:
- **Cross-vendor review** — the reviewer is never the same vendor as the implementer.
- **Verify, don't trust** — re-run tests / read the diff / look at the app before "done".

Skills: `launch-subagent`, `stack-a-grok-host`, `grok-subagent`, `sol-review`, `opus-review`, `luna-subagent`.

| Mode | When | Orchestrator | Implementer | Reviewer |
|------|------|--------------|-------------|----------|
| **Normal** | Claude Code, Opus budget OK | Opus 5 | `grok-subagent` | Sol → Terra /high |
| **Opus-Saver** | Opus usage ≈ 70%+ | Sonnet 5 (`/model sonnet`) | `grok-subagent` | Sol → Terra /high |
| **Grok-host** | Opus at session limit / Grok TUI | This Grok session | **this session** (do not spawn `grok-subagent`) | Sol → Terra /high |

**Invariant:** implementer is always Grok; reviewer is always a non-Grok GPT.

## Pipeline

1. Host plans + brief
2. Grok implements (`grok-subagent` from Claude; in-session on Grok-host)
3. **Sol → Terra /high** reviews — show the report verbatim
4. Grok fixes
5. Host verifies independently. High-risk sign-off waits for Opus.
6. Sonnet (or Grok-host) for commit / merge / push when the user asks

## Model availability — re-probe each session

- `grok` CLI → Grok 4.5.
- `codex` CLI → `gpt-5.6-sol`, `gpt-5.6-terra` (fallback reviewer at `high`), `gpt-5.6-luna`, `gpt-5.5`.
- Sol has flipped available/unavailable on a ChatGPT-account Codex login. Probe first.

## Rules

1. Default coder is **Grok 4.5**. Not Sonnet, Luna, or local.
2. Reviewer ≠ implementer vendor. Never let Grok review Grok.
3. Verify agent claims yourself (tests, diff, running app).
4. Subagents start blind — full brief in a file.
5. Auto-delegate coding from a Claude host. On Grok-host, implement here.
6. Never print or commit an API key. Load via `source ~/.config/stack-a/env`.

Full policy: `~/Documents/My_skills/STACK-A.md` and https://github.com/pgm-norbert/My_skills
