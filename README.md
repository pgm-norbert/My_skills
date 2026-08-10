# My_skills (pgm-norbert)

Fork / working copy of davidondrej-style Agent Skills, with a personal
**Stack A** orchestration policy (Opus orchestrates, Grok implements, GPT-5.6 Sol reviews).

## Stack A (quality-first)

See **[STACK-A.md](./STACK-A.md)** for the full guide. The canonical policy lives
in `~/.claude/CLAUDE.md`; `CLAUDE.global.snippet.md` here is generated from it.

| Role | Model |
|------|--------|
| Orchestrator | **Opus 5** (host session) |
| Senior engineer (most coding) | **Grok 4.5** (`grok-subagent`) |
| Junior worker | **GPT-5.6 Luna** (`luna-subagent`) — bounded mechanical work |
| Reviewer | **GPT-5.6 Sol**, falls back to GPT-5.5 / Terra (`sol-review`) |
| Light ops | **Claude Sonnet 5** (commit / merge / push / PR) |
| Final sign-off | **Opus 5** (`opus-review`) |

Pipeline: **Opus plan → Grok implement → Sol review/fix → Opus sign-off → Sonnet for ship ceremony**.

The reviewer is deliberately a **different vendor** from the implementer. Never let
a model review its own work.

Worker lanes:

- **Grok 4.5** — default implementer: features, multi-file fixes, tests, refactors.
- **GPT-5.6 Luna** (`gpt-5.6-luna`) — bulk edits, classification, short mechanical
  tasks; not the default feature implementer.

Key skills:

- `skills/agent-orchestration/launch-subagent/` — model policy
- `skills/agent-orchestration/grok-subagent/` — default implementer
- `skills/agent-orchestration/luna-subagent/` — bounded mechanical work
- `skills/agent-orchestration/sol-review/` — adversarial review (cross-vendor)
- `skills/agent-orchestration/opus-review/` — final sign-off
- Host **Sonnet 5** — light ops only (not feature coding)
- `skills/agent-orchestration/deepseek-subagent/` — **DEPRECATED**, DeepSeek retired 2026-08-07

Credentials: the `grok` and `codex` CLIs authenticate themselves; anything else goes
in `~/.config/stack-a/env` (never commit API keys).

## Upstream layout

Skills are grouped into category folders under `skills/`. Each skill lives in
its own folder and starts with a `SKILL.md` file that explains when and how to
use it.

- `skills/agent-orchestration/` — Running, scheduling, delegating to, and coordinating AI coding agents, including agent-to-agent workflows, agent loops, and agent benchmarks.
- `skills/skill-authoring/` — Creating, improving, distributing, and publishing Agent Skills and agent context files.
- `skills/research-and-web/` — Finding and pulling information from the web, research APIs, browsers, and YouTube.
- `skills/thinking-and-docs/` — Structured thinking, interviewing, teaching, and turning ideas into clear documentation.
- `skills/ops-and-setup/` — Machine, server, security, and tool setup, configuration, and operations.

## Global Claude Code policy

Install Stack A into every Claude Code session by keeping the block in
`~/.claude/CLAUDE.md` (see `CLAUDE.global.snippet.md` for the canonical copy).
