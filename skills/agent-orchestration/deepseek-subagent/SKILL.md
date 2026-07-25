---
name: deepseek-subagent
description: 'Launch DeepSeek V4 Pro as Stack A junior engineer (default implementer — does most coding work). Preferred harness: Hermes Agent (already on this machine). Use when the orchestrator (Opus 5) delegates a self-contained coding task with a complete brief. After DeepSeek, route to Grok senior review then Opus sign-off.'
disable-model-invocation: true
---

# DeepSeek V4 Pro as Junior Engineer (Stack A)

DeepSeek is Stack A’s **default implementer** (junior engineer). It does **most of the work** from a complete brief.

**On this machine the coding harness is Hermes Agent** (`hermes` CLI) — not Aider, not host Task model-routing. Hermes already supports `--provider deepseek` and tool-calling (edit files, run shell, etc.).

Senior review/fix: **Grok 4.5**. Final sign-off: **Opus 5**.

## When to delegate

- Self-contained coding task with clear success criteria (feature, fix, tests, refactor, investigate).
- Bulk implementation while Opus stays free to plan and sign off.
- Parallel independent tasks (prefer `--worktree` / one cwd ownership set per run).

Do NOT use DeepSeek as sole merge-gate reviewer or final sign-off.

## Credentials (never commit / never echo)

```bash
# Stack A env (optional for Hermes — Hermes also reads ~/.hermes/.env)
source "$HOME/.config/stack-a/env" 2>/dev/null || true

# Hermes must have DEEPSEEK_API_KEY in ~/.hermes/.env (already configured on this machine).
# Verify without printing the key:
rg -q '^DEEPSEEK_API_KEY=.' "$HOME/.hermes/.env" && echo "Hermes DeepSeek key: present"
```

- Preferred model slug: **`deepseek-v4-pro`** (API also offers `deepseek-v4-flash`).
- Stack A env default: `DEEPSEEK_MODEL=deepseek-v4-pro`.
- **Never** paste keys into prompts, commits, skill files, or chat logs.

## Preflight

```bash
which hermes
rg -q '^DEEPSEEK_API_KEY=.' "$HOME/.hermes/.env" || echo "MISSING: add DEEPSEEK_API_KEY to ~/.hermes/.env"
# optional smoke (should print DEEPSEEK_OK):
# hermes chat -Q --yolo --provider deepseek -m deepseek-v4-pro -q "Reply with exactly: DEEPSEEK_OK"
```

## Preferred launch — Hermes (default on this machine)

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
# Non-interactive junior implementer:
hermes chat -Q --yolo \
  --provider deepseek \
  -m deepseek-v4-pro \
  --max-turns 40 \
  -q "$(cat "$PROMPT_FILE")" \
  > "$OUT" 2>/tmp/deepseek-err.log

# Or pass a long brief via multiple -q is not ideal — prefer one -q with full text.
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

### Flags that matter

| Flag | Meaning |
|------|---------|
| `--provider deepseek` | Use DeepSeek API (not NVIDIA/Ollama default) |
| `-m deepseek-v4-pro` | Junior model (use `deepseek-v4-flash` only if user wants cheaper/faster) |
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

1. Read `$OUT` + `git status` / diff.
2. Run verification commands (orchestrator).
3. **Required for non-trivial work:** `grok-review` → fix via `grok-subagent` if needed.
4. **Final:** Opus sign-off (`opus-review`).

## Failure modes

| Symptom | Fix |
|---------|-----|
| Hermes uses wrong provider/model | Pass `--provider deepseek -m deepseek-v4-pro` explicitly |
| Auth / 401 | `DEEPSEEK_API_KEY` missing from `~/.hermes/.env` |
| Model not found | Use exact slugs from API: `deepseek-v4-pro` or `deepseek-v4-flash` |
| Hangs / tool prompts | Add `--yolo` on trusted repos; raise `--max-turns` |
| “No harness” | Install Hermes (`hermes`) or Aider — do not hand-roll agents mid-task |

## Rules

- One task per launch. Split big jobs.
- Junior only — never final sign-off.
- Never log or commit API keys.
- Default route when user says “use Stack A and delegate”: **this skill (Hermes + DeepSeek)**, not Ollama, not Grok-as-implementer.

## Relationship to other skills

| Skill | Role |
|-------|------|
| `launch-subagent` | Stack A policy |
| `deepseek-subagent` | Junior implementer (this file) |
| `grok-review` | Senior review |
| `grok-subagent` | Senior fix / hard rescue |
| `opus-review` | Final sign-off |
