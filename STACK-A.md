# Stack A — Quality-first orchestration

Personal policy: **Opus orchestrates and signs off**, **DeepSeek does most of the coding**, **Grok is the senior engineer** who reviews and fixes.

## Roles

| Role | Model | Job |
|------|--------|-----|
| **Orchestrator** | **Opus 5** | Host session. Plans, writes briefs, routes work, integrates. Does **not** grind implementation by default. |
| **Junior engineer** (default implementer) | **DeepSeek V4 Pro** | Does **most of the work**: features, fixes, tests from a complete brief. Skill: `deepseek-subagent`. |
| **Senior engineer** | **Grok 4.5** | Reviews junior work, **fixes** serious issues, hard problems the junior can’t land. Skills: `grok-review` (review), `grok-subagent` (fix / hard rescue). |
| **Final sign-off** | **Opus 5** | Last gate before “done” / merge claim. Skill: `opus-review` (or host Opus verification + sign-off). |

## Default delivery pipeline

```
User
 └─ Opus 5 (orchestrator): plan + brief + definition of done
      └─ DeepSeek V4 Pro (junior): implement most of the task
      └─ Grok 4.5 (senior): review; fix Critical/Important issues
      └─ Opus 5 (sign-off): final check of diff + tests; report to user
```

1. **Opus** writes a self-contained brief (goal, paths, constraints, verify commands).
2. **DeepSeek** implements against that brief (`deepseek-subagent`).
3. **Grok** reviews the diff; if issues, **Grok fixes** (or re-briefs DeepSeek for tiny follow-ups — prefer Grok for fixes on code it just reviewed).
4. **Opus** re-reads summary + `git diff` + test output and **signs off** (or sends one more fix cycle). Never claim done without Opus sign-off when this pipeline ran.

## Credentials (DeepSeek)

- **Never** put API keys in skill markdown or git.
- Local file: `~/.config/stack-a/env` (mode `600`), sourced from `~/.zshrc`.
- Variables: `DEEPSEEK_API_KEY`, `DEEPSEEK_BASE_URL` (default `https://api.deepseek.com`), `DEEPSEEK_MODEL` (default `deepseek-chat` for V4 Pro API access).

```bash
source ~/.config/stack-a/env
# key loaded? yes if non-empty — do not echo the value
test -n "$DEEPSEEK_API_KEY" && echo "DeepSeek ready ($DEEPSEEK_MODEL)"
```

## Local Ollama — not the default path

**Stack A does not mean “use local models.”** Ollama being installed is irrelevant unless the user asks for local.

| User says | Route |
|-----------|--------|
| “use Stack A and delegate …” | **DeepSeek V4 Pro** junior → Grok senior → Opus sign-off |
| “use local / Ollama / qwen …” | Only then consider local Qwen |
| DeepSeek API down + non-trivial task | **Grok 4.5**, not Ollama |
| DeepSeek down + tiny mechanical only | Optional: Qwen3.6 → else `qwen3.5:9b` |

Do **not** open with `ollama list` or “which local model” when Stack A was requested.

## Banned as defaults

- **Ollama / local models** as the first pick under Stack A.
- **Codex / GPT Sol** — only if the user explicitly asks (`codex-subagent`).
- **Composer / Sonnet** — not the default implementer under Stack A.
- Skipping Grok senior review on non-trivial DeepSeek output.
- Claiming merge-ready without **Opus** sign-off after the pipeline.
- Briefing **Grok first** for bulk work when DeepSeek is the junior (Grok is senior review/fix, not default implementer).

## Skills map

| Skill | Model / role |
|-------|----------------|
| `launch-subagent` | Policy (this stack) |
| `deepseek-subagent` | Junior implementer (DeepSeek V4 Pro) |
| `grok-subagent` | Senior fix / hard rescue (Grok 4.5) |
| `grok-review` | Senior review of junior work (Grok 4.5) |
| `opus-review` | Final sign-off (Opus 5) |
| `fable-review` | Alias → `opus-review` |
| `gpt-review` | Alias → `grok-review` (Grok senior, not OpenAI) |
| `codex-subagent` | Legacy — user must ask |

## Install / sync

Skills live under `skills/agent-orchestration/`. Symlink or copy into agent skill roots (`~/.agents/skills`, `~/.claude/skills`, etc.).
