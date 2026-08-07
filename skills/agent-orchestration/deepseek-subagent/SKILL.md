---
name: deepseek-subagent
description: 'Launch DeepSeek as Stack A junior engineer. Default coding model is DeepSeek V4 Pro (Hermes implementer — most coding work). DeepSeek V4 Flash is the fast junior / chat lane (RAG, short edits, product chat) — not the default feature implementer. Use when the orchestrator (Opus 5) delegates a self-contained coding task with a complete brief. After Pro coding, route to Grok senior review then Opus sign-off.'
disable-model-invocation: true
---

> **DEPRECATED 2026-08-07 — DeepSeek is retired from Stack A.**
> The default implementer is now **Grok 4.5** (`grok-subagent`); bounded mechanical
> work goes to **GPT-5.6 Luna** (`luna-subagent`); review is **GPT-5.6 Sol**
> (`sol-review`). Kept for reference only — do not route new work here.


# DeepSeek as Junior Engineer (Stack A)

DeepSeek is Stack A’s **default implementer** family. Two lanes:

| Lane | Slug | Role |
|------|------|------|
| **Pro** (default coding junior) | `deepseek-v4-pro` | Hermes implementer for features, fixes, tests, refactors |
| **Flash** (fast junior / chat) | `deepseek-v4-flash` | Product chat, RAG Q&A, short mechanical edits, classification |

**On this machine the coding harness is Hermes Agent** (`hermes` CLI) — not Aider, not host Task model-routing. Hermes already supports `--provider deepseek` and tool-calling (edit files, run shell, etc.).

Senior review/fix: **Grok 4.5**. Final sign-off: **Opus 5**. Light ops: **Sonnet 5**.

## When to use Pro vs Flash

### Pro — default for coding delegation

- Self-contained coding task with clear success criteria (feature, fix, tests, refactor, investigate).
- Bulk implementation while Opus stays free to plan and sign off.
- Parallel independent tasks (prefer `--worktree` / one cwd ownership set per run).
- User says “use Stack A and delegate” for **code**.

### Flash — fast junior / chat only

- In-app chat and RAG answer generation (prefer Flash in product LLM clients).
- Short one-file mechanical edits when explicitly requested or clearly tiny.
- Classification / labeling / high-volume cheap inference.
- User says “use Flash”, “cheap”, or “fast chat”.

Do **not** silently pick Flash for multi-file features to save money. Do **not** use either DeepSeek model as sole merge-gate reviewer or final sign-off.

## Credentials (never commit / never echo)

```bash
# Stack A env (optional for Hermes — Hermes also reads ~/.hermes/.env)
source "$HOME/.config/stack-a/env" 2>/dev/null || true

# Hermes must have DEEPSEEK_API_KEY in ~/.hermes/.env (already configured on this machine).
# Verify without printing the key:
rg -q '^DEEPSEEK_API_KEY=.' "$HOME/.hermes/.env" && echo "Hermes DeepSeek key: present"
```

- Coding default: **`deepseek-v4-pro`** (`DEEPSEEK_MODEL` in stack-a env).
- Flash: **`deepseek-v4-flash`** (optional `DEEPSEEK_FLASH_MODEL` for product apps).
- **Never** paste keys into prompts, commits, skill files, or chat logs.

## Preflight

```bash
which hermes
rg -q '^DEEPSEEK_API_KEY=.' "$HOME/.hermes/.env" || echo "MISSING: add DEEPSEEK_API_KEY to ~/.hermes/.env"
# optional smoke (should print DEEPSEEK_OK):
# hermes chat -Q --yolo --provider deepseek -m deepseek-v4-pro -q "Reply with exactly: DEEPSEEK_OK"
```

## Preferred launch — Hermes Pro (default coding junior)

Write a full brief to a file, then run Hermes **non-interactive** from the **repo root** (`--yolo` auto-approves tools for trusted user-owned repos only):

```bash
REPO=/path/to/repo
PROMPT_FILE=$(mktemp /tmp/deepseek-prompt.XXXXXX)
OUT=$(mktemp /tmp/deepseek-out.XXXXXX)

cat > "$PROMPT_FILE" <<'EOF'
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
EOF

cd "$REPO"
# Non-interactive junior implementer (Pro):
hermes chat -Q --yolo \
  --provider deepseek \
  -m deepseek-v4-pro \
  --max-turns 40 \
  -q "$(cat "$PROMPT_FILE")" \
  > "$OUT" 2>/tmp/deepseek-err.log

cat "$OUT"
git -C "$REPO" status --short
```

### Background (orchestrator polls)

```bash
nohup hermes chat -Q --yolo \
  --provider deepseek \
  -m deepseek-v4-pro \
  --max-turns 40 \
  -q "$(cat "$PROMPT_FILE")" \
  > "$OUT" 2>/tmp/deepseek-err.log &
echo "PID=$! OUT=$OUT"
```

### Flash launch (only when appropriate)

Same Hermes shape, different model — tiny briefs, user asked for Flash, or chat-style work:

```bash
hermes chat -Q --yolo \
  --provider deepseek \
  -m deepseek-v4-flash \
  --max-turns 20 \
  -q "$(cat "$PROMPT_FILE")" \
  > "$OUT" 2>/tmp/deepseek-err.log
```

For **product apps**, prefer calling the OpenAI-compatible DeepSeek API with `model: deepseek-v4-flash` (or env `DEEPSEEK_FLASH_MODEL`) rather than spinning Hermes for every chat turn.

### Flags that matter

| Flag | Meaning |
|------|---------|
| `--provider deepseek` | Use DeepSeek API (not NVIDIA/Ollama default) |
| `-m deepseek-v4-pro` | **Default coding junior** |
| `-m deepseek-v4-flash` | **Fast junior / chat** — only when lane matches |
| `-q "..."` | Single-shot non-interactive query |
| `-Q` | Quiet: final answer + session id |
| `--yolo` | Auto-approve tools (trusted repos only) |
| `--worktree` / `-w` | Isolated git worktree for parallel juniors |
| `--max-turns N` | Cap tool loop |

DeepSeek via Hermes sees **nothing** of the orchestrator chat — put everything in `-q` / the brief file.

## Fallback harnesses (only if Hermes missing)

1. **Aider** (install if needed): `uv tool install aider-chat` then `aider --model deepseek/deepseek-v4-pro` with `DEEPSEEK_API_KEY` set.
2. **Host Task targeting DeepSeek** — only if the host can select DeepSeek (most cannot).
3. Do **not** invent a custom agent loop mid-task if Hermes/Aider are unavailable — report **BLOCKED: no coding harness** and ask the user to install Hermes or Aider.

## After DeepSeek finishes

**After Pro coding:**

1. Read `$OUT` + `git status` / diff.
2. Run verification commands (orchestrator).
3. **Required for non-trivial work:** `grok-review` → fix via `grok-subagent` if needed.
4. **Final:** Opus sign-off (`opus-review`).

**After Flash chat / short edit:** verify if files changed; no full senior pipeline unless the edit was non-trivial.

## Failure modes

| Symptom | Fix |
|---------|-----|
| Hermes uses wrong provider/model | Pass `--provider deepseek -m deepseek-v4-pro` (or flash) explicitly |
| Auth / 401 | `DEEPSEEK_API_KEY` missing from `~/.hermes/.env` |
| Model not found | Use exact slugs: `deepseek-v4-pro` or `deepseek-v4-flash` |
| Hangs / tool prompts | Add `--yolo` on trusted repos; raise `--max-turns` |
| “No harness” | Install Hermes (`hermes`) or Aider — do not hand-roll agents mid-task |
| Feature coded on Flash poorly | Re-run on **Pro** with a full brief |

## Rules

- One task per launch. Split big jobs.
- **Pro** = default coding junior; **Flash** = fast/chat lane.
- Junior only — never final sign-off.
- Never log or commit API keys.
- Default route when user says “use Stack A and delegate” (code): **Hermes + Pro**, not Flash, not Ollama, not Grok-as-implementer.

## Relationship to other skills

| Skill | Role |
|-------|------|
| `launch-subagent` | Stack A policy (Pro vs Flash table) |
| `deepseek-subagent` | Junior implementer (this file) |
| `grok-review` | Senior review |
| `grok-subagent` | Senior fix / hard rescue |
| `opus-review` | Final sign-off |
| Sonnet 5 host | Light ops (commit / merge / push) only |
