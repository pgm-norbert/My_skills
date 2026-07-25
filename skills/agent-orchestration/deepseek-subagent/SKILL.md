---
name: deepseek-subagent
description: 'Launch DeepSeek V4 Pro as Stack A junior engineer (default implementer — does most coding work). Use when the orchestrator (Opus 5) delegates a self-contained coding task with a complete brief. After DeepSeek, route to Grok senior review then Opus sign-off.'
disable-model-invocation: true
---

# DeepSeek V4 Pro as Junior Engineer (Stack A)

DeepSeek is Stack A’s **default implementer** (junior engineer). It does **most of the work** from a complete brief. Auth is the user’s DeepSeek API key via local env — **never print the key**.

Senior review/fix is **Grok 4.5**. Final sign-off is **Opus 5**.

## When to delegate

- Self-contained coding task with clear success criteria (feature, fix, tests, refactor).
- Bulk implementation while Opus stays free to plan and sign off.
- Parallel independent tasks (one worktree / ownership set per run).

Do NOT delegate tasks that need conversation context you can’t fully write into the prompt.
Do NOT use DeepSeek as sole merge-gate reviewer or final sign-off.

## Credentials (never commit / never echo)

```bash
# Preferred: local Stack A env (created on the user’s machine)
source "$HOME/.config/stack-a/env"
test -n "$DEEPSEEK_API_KEY" || { echo "MISSING DEEPSEEK_API_KEY — set ~/.config/stack-a/env"; exit 1; }

# Defaults if unset:
export DEEPSEEK_BASE_URL="${DEEPSEEK_BASE_URL:-https://api.deepseek.com}"
export DEEPSEEK_MODEL="${DEEPSEEK_MODEL:-deepseek-chat}"   # V4 Pro API access; override if account uses another slug
```

- Key file path: `~/.config/stack-a/env` (mode `600`).
- **Never** paste the key into prompts, commits, skill files, or chat logs.
- If key missing → stop and tell the user to configure `~/.config/stack-a/env`.

## Preflight

```bash
source "$HOME/.config/stack-a/env" 2>/dev/null || true
test -n "$DEEPSEEK_API_KEY" && echo "DeepSeek key: present"
echo "model=${DEEPSEEK_MODEL:-deepseek-chat} base=${DEEPSEEK_BASE_URL:-https://api.deepseek.com}"
# OpenAI-compatible smoke (no key in output):
curl -sS -o /tmp/ds-models.json -w "%{http_code}" \
  -H "Authorization: Bearer $DEEPSEEK_API_KEY" \
  "${DEEPSEEK_BASE_URL:-https://api.deepseek.com}/models" | tail -1
# expect 200
```

## Launch patterns

DeepSeek exposes an **OpenAI-compatible** HTTP API. Prefer a coding-agent harness that can target a custom base URL. Pick the first available:

### A) Aider (if installed)

```bash
source "$HOME/.config/stack-a/env"
OUT=$(mktemp /tmp/deepseek-out.XXXXXX)
PROMPT_FILE=$(mktemp /tmp/deepseek-prompt.XXXXXX)
# write full brief into $PROMPT_FILE

cd /path/to/repo
aider --yes \
  --model "openai/${DEEPSEEK_MODEL}" \
  --openai-api-base "$DEEPSEEK_BASE_URL" \
  --openai-api-key "$DEEPSEEK_API_KEY" \
  --message-file "$PROMPT_FILE" \
  > "$OUT" 2>/tmp/deepseek-err.log
```

### B) Host Task / agent with DeepSeek model

If the host product can run a subagent on **DeepSeek** (or OpenAI-compatible with base URL + key), use that with the same full brief. Still treat it as **junior** — Grok review + Opus sign-off after.

### C) OpenAI-compatible CLI (`openai` / custom)

```bash
source "$HOME/.config/stack-a/env"
export OPENAI_API_KEY="$DEEPSEEK_API_KEY"
export OPENAI_BASE_URL="$DEEPSEEK_BASE_URL"
# Then run whatever coding agent supports OPENAI_BASE_URL + OPENAI_API_KEY
# with model $DEEPSEEK_MODEL
```

### Brief template (always)

```markdown
# Task
(goal, constraints, files to touch, definition of done)

# Context
(paths, invariants, what NOT to change)

# Verify
(exact commands: tests, typecheck)

# Return
- Short summary of what you changed
- Commands you ran and results
- Anything uncertain
```

DeepSeek sees **nothing** of the orchestrator conversation. Put everything in the brief.

## After DeepSeek finishes

1. Read the summary + `git -C /path/to/repo status` and diff.
2. Run verification commands yourself (orchestrator).
3. **Required next step for non-trivial work:** `grok-review` (senior), then fix via `grok-subagent` if needed.
4. **Final:** Opus sign-off (`opus-review` / host Opus) before claiming done.

## Failure modes

| Symptom | Fix |
|---------|-----|
| 401 / auth | Key wrong or missing in `~/.config/stack-a/env` |
| Model not found | Adjust `DEEPSEEK_MODEL` (try `deepseek-chat` / account-specific slug) |
| No coding harness | Install aider or use host Task; don’t invent a half-baked agent loop mid-task |
| Rate limit | Report to user; don’t tight-loop |

## Rules

- One task per launch. Split big jobs.
- Junior only — never final sign-off.
- Never log or commit `DEEPSEEK_API_KEY`.

## Relationship to other skills

| Skill | Role |
|-------|------|
| `launch-subagent` | Stack A policy |
| `deepseek-subagent` | Junior implementer (this file) |
| `grok-review` | Senior review |
| `grok-subagent` | Senior fix / hard rescue |
| `opus-review` | Final sign-off |
