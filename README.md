# My_skills (pgm-norbert)

Fork / working copy of davidondrej-style Agent Skills, with a personal
**Stack A** orchestration policy (Opus + Grok, not OpenAI-default).

## Stack A (quality-first)

See **[STACK-A.md](./STACK-A.md)** for the full guide.

| Role | Model |
|------|--------|
| Orchestrator | **Opus 5** (host session) |
| Hard executor | **Grok 4.5** (`grok-subagent` CLI) |
| Routine executor | **DeepSeek V4** / **Qwen3.6 local** |
| Reviewer | **Opus 5** *or* **Grok 4.5** (opposite of implementer) |

Key skills:

- `skills/agent-orchestration/launch-subagent/` — hard model policy
- `skills/agent-orchestration/grok-subagent/` — hard executor via Grok CLI
- `skills/agent-orchestration/opus-review/` / `grok-review/` — opposite-family review
- `fable-review` / `gpt-review` — compat aliases → opus / grok review
- `codex-subagent` — **legacy** OpenAI path only if you explicitly ask

## Upstream layout

Skills are grouped into category folders under `skills/`. Each skill lives in
its own folder and starts with a `SKILL.md` file that explains when and how to
use it.

- `skills/agent-orchestration/` — Running, scheduling, delegating to, and coordinating AI coding agents, including agent-to-agent workflows, agent loops, and agent benchmarks.
- `skills/skill-authoring/` — Creating, improving, distributing, and publishing Agent Skills and agent context files.
- `skills/research-and-web/` — Finding and pulling information from the web, research APIs, browsers, and YouTube.
- `skills/thinking-and-docs/` — Structured thinking, interviewing, teaching, and turning ideas into clear documentation.
- `skills/ops-and-setup/` — Machine, server, security, and tool setup, configuration, and operations.
