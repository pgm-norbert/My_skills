---
name: grok-subagent
description: Launch xAI Grok CLI as a hard-executor subagent (Stack A default implementer = Grok 4.5). Use when the orchestrator (usually Opus 5) delegates a self-contained coding task — features, fixes, refactors, parallel worktrees. Prefer this over Codex for Stack A.
disable-model-invocation: true
---

# Grok CLI as a Hard Executor (Stack A)

Grok CLI is xAI’s terminal coding agent. Headless mode (`-p` / `--prompt-file`) runs a **separate OS process**: full tool loop, own context, writes files in `--cwd`, then prints a final answer and exits. Auth is the user’s Grok login — never print credentials.

This is Stack A’s **default hard executor**. The host agent (ideally **Opus 5**) stays orchestrator.

## When to delegate

- Self-contained coding task with clear success criteria (fix, feature, refactor).
- Parallel independent tasks (one worktree / ownership set per Grok process).
- Implementation while Opus stays free to plan, integrate, and review.

Do NOT delegate tasks that need conversation context you can’t fully write into the prompt.

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

# Return
- Short summary of what you changed
- Commands you ran and results
- Anything uncertain
EOF

# Foreground (orchestrator waits):
grok -p "$(cat "$PROMPT_FILE")" \
  --cwd /path/to/repo \
  -m grok-4.5 \
  --always-approve \
  --output-format plain \
  > "$OUT" 2>/tmp/grok-err.XXXXXX

# Or long prompt via file (if shell arg length is an issue):
# grok --prompt-file "$PROMPT_FILE" --cwd /path/to/repo -m grok-4.5 --always-approve
```

### Background from Claude Code / host shell

Runs as a **background shell job** (still a separate Grok CLI process — not inside Claude):

```bash
nohup grok --prompt-file "$PROMPT_FILE" \
  --cwd /path/to/repo \
  -m grok-4.5 \
  --always-approve \
  --output-format plain \
  > "$OUT" 2>/tmp/grok-err.log &
echo "GROK_PID=$! OUT=$OUT"
# Poll: tail -20 "$OUT"; wait $GROK_PID; cat "$OUT"
```

Host tools that support `run_in_background` / background Bash are fine — same idea.

### Interactive watch (optional)

```bash
# New terminal / cmux pane the user can watch
grok --cwd /path/to/repo -m grok-4.5 --always-approve "Full task prompt…"
```

## Model rules

- Always use **Grok 4.5** (`-m grok-4.5`) unless the user names another Grok model.
- Grok sees **nothing** of the orchestrator’s conversation. Put all context in the prompt file.
- After exit: read `$OUT`, then `git -C /path/to/repo status --short` and relevant diffs.
- Orchestrator runs tests / typecheck; do not trust “done” without evidence.

## Parallel runs

One worktree (or disjoint file ownership) per Grok process:

```bash
git worktree add /tmp/wt-taskA -b grok/task-a
grok --prompt-file /tmp/taskA.md --cwd /tmp/wt-taskA -m grok-4.5 --always-approve
```

## Failure modes

- Empty / hung output → check `/tmp/grok-err*`, auth, and that `--cwd` exists.
- Permission prompts → pass `--always-approve` (only on trusted repos the user owns).
- Rate limits → report to user; do not tight-loop retry.
- Model not found → run `grok models` and pick the closest available Grok 4.x slug.

## Rules

- One task per launch. Split big jobs.
- Orchestrator reviews the diff before declaring the task done.
- After Grok implements, prefer **`opus-review`** (opposite-family reviewer).

## Relationship to other skills

| Skill | Role |
|-------|------|
| `launch-subagent` | Stack A policy |
| `grok-subagent` | Hard executor (this file) |
| `opus-review` | Review after Grok implements |
| `grok-review` | Review when Opus implemented |
| `codex-subagent` | Legacy OpenAI path — only if user asks |
