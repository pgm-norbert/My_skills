---
name: grok-subagent
description: 'Launch xAI Grok 4.5 as Stack A senior engineer — review fixes, hard rescue, and non-trivial repair after DeepSeek junior work. Prefer deepseek-subagent for default implementation bulk.'
disable-model-invocation: true
---

# Grok CLI as Senior Engineer (Stack A)

Grok CLI is xAI’s terminal coding agent. Headless mode (`-p` / `--prompt-file`) runs a **separate OS process**: full tool loop, own context, writes files in `--cwd`, then prints a final answer and exits. Auth is the user’s Grok login — never print credentials.

Under Stack A, Grok is the **senior engineer**:

- Reviews junior (DeepSeek) work and **implements fixes** for serious issues.
- Hard rescue when DeepSeek is blocked (security, multi-system design, thrashing).
- **Not** the default bulk implementer — that is **DeepSeek V4 Pro** (`deepseek-subagent`).

Orchestrator + final sign-off remain **Opus 5**.

## When to delegate

- Fix Critical/Important findings after `grok-review` (or after Opus notes issues on junior output).
- Hard problem DeepSeek failed or should not own (security, auth, data loss risk).
- User explicitly asks for Grok to implement.

Do NOT use Grok as the default “do everything” implementer when DeepSeek can take the bulk.

## Preflight

```bash
which grok || which grok   # expect ~/.grok/bin/grok or ~/.local/bin/grok
grok models                # expect grok-4.5 listed; default may already be grok-4.5
```

Not authenticated → tell the user to log in via `grok` once. Never read or print auth tokens.

## Launch (preferred — headless, capturable)

```bash
OUT=$(mktemp /tmp/grok-out.XXXXXX)
PROMPT_FILE=$(mktemp /tmp/grok-prompt.XXXXXX)
cat > "$PROMPT_FILE" <<'EOF'
# Task
(goal, constraints, files to touch, definition of done, verification commands)

# Context
(paths, invariants, what NOT to change)
# If fixing junior work: paste review findings verbatim

# Return
- Short summary of what you changed
- Commands you ran and results
- Anything uncertain
EOF

grok -p "$(cat "$PROMPT_FILE")" \
  --cwd /path/to/repo \
  -m grok-4.5 \
  --always-approve \
  --output-format plain \
  > "$OUT" 2>/tmp/grok-err.XXXXXX

# Or long prompt via file:
# grok --prompt-file "$PROMPT_FILE" --cwd /path/to/repo -m grok-4.5 --always-approve
```

### Background from Claude Code / host shell

```bash
nohup grok --prompt-file "$PROMPT_FILE" \
  --cwd /path/to/repo \
  -m grok-4.5 \
  --always-approve \
  --output-format plain \
  > "$OUT" 2>/tmp/grok-err.log &
echo "GROK_PID=$! OUT=$OUT"
```

## Model rules

- Always use **Grok 4.5** (`-m grok-4.5`) unless the user names another Grok model.
- Grok sees **nothing** of the orchestrator’s conversation. Put all context in the prompt file.
- After exit: read `$OUT`, then `git -C /path/to/repo status --short` and relevant diffs.
- Orchestrator runs tests / typecheck; do not trust “done” without evidence.
- After Grok fixes: **Opus final sign-off** (`opus-review`).

## Parallel runs

One worktree (or disjoint file ownership) per Grok process.

## Failure modes

- Empty / hung output → check `/tmp/grok-err*`, auth, and that `--cwd` exists.
- Permission prompts → pass `--always-approve` (only on trusted repos the user owns).
- Rate limits → report to user; do not tight-loop retry.
- Model not found → run `grok models` and pick the closest available Grok 4.x slug.

## Relationship to other skills

| Skill | Role |
|-------|------|
| `launch-subagent` | Stack A policy |
| `deepseek-subagent` | Junior implementer (most work) |
| `grok-subagent` | Senior fix / hard rescue (this file) |
| `grok-review` | Senior review report |
| `opus-review` | Final sign-off |
| `codex-subagent` | Legacy OpenAI path — only if user asks |
