# My_skills (pgm-norbert)

Fork / working copy of davidondrej-style Agent Skills, with a personal
**Stack A** orchestration policy (Opus + DeepSeek + Grok, not OpenAI-default).

## Stack A (quality-first)

See **[STACK-A.md](./STACK-A.md)** for the full guide.

| Role | Model |
|------|--------|
| Orchestrator | **Opus 5** (host session) |
| Junior engineer (most coding) | **DeepSeek V4 Pro** (`deepseek-subagent`) |
| Fast junior / chat | **DeepSeek V4 Flash** (RAG, product chat, short edits) |
| Senior engineer | **Grok 4.5** (`grok-review` + `grok-subagent` fixes) |
| Light ops | **Claude Sonnet 5** (commit / merge / push / PR) |
| Final sign-off | **Opus 5** (`opus-review`) |

Pipeline: **Opus plan → DeepSeek Pro implement → Grok review/fix → Opus sign-off → Sonnet for ship ceremony**.

DeepSeek lanes:

- **Pro** (`deepseek-v4-pro`) — default coding junior via Hermes.
- **Flash** (`deepseek-v4-flash`) — fast junior / chat / RAG; not the default feature implementer.

Key skills:

- `skills/agent-orchestration/launch-subagent/` — model policy
- `skills/agent-orchestration/deepseek-subagent/` — junior implementer (Pro default; Flash optional)
- `skills/agent-orchestration/grok-subagent/` — senior fixes / hard rescue
- `skills/agent-orchestration/grok-review/` / `opus-review/` — senior review / final sign-off
- Host **Sonnet 5** — light ops only (not feature coding)
- `fable-review` / `gpt-review` — aliases → opus sign-off / grok senior review
- `codex-subagent` — **legacy** OpenAI path only if you explicitly ask

DeepSeek credentials: local only `~/.config/stack-a/env` (never commit API keys).

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
