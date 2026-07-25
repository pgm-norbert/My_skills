---
name: grok-review
description: 'Launch Grok 4.5 as Stack A senior engineer for a deep review of junior (DeepSeek) work, or when the user says "/grok-review". After review, use grok-subagent to fix serious issues. Final sign-off remains Opus 5.'
---

# Grok Review — Senior Engineer (Stack A)

Launch **Grok 4.5** to review work fully and carefully, as a **senior engineer** reviewing a **junior** (usually DeepSeek V4 Pro).

**When to use:**

- After **DeepSeek** (or any junior) implements a non-trivial task.
- When the user says `/grok-review` / “grok review”.
- Optionally after Opus self-implemented something and wants a senior pass (still Grok).

**Not** final product sign-off — that is **Opus 5** (`opus-review`).

## How to launch

Prefer headless Grok CLI (separate process):

```bash
OUT=$(mktemp /tmp/grok-review.XXXXXX)
PROMPT_FILE=$(mktemp /tmp/grok-review-prompt.XXXXXX)
cat > "$PROMPT_FILE" <<'EOF'
You are a senior engineer reviewing a junior's work. Be neutral. Do not redesign for taste.

## Scope
(paths / PR / branch / what changed)
(note: implementer was DeepSeek V4 Pro junior unless stated otherwise)

## What to do
- Go deep: correctness, edge cases, security, missing tests, merge risk
- Separate: Critical / Important / Minor
- For Critical/Important: say how to fix (concrete)
- Do not implement fixes in this pass unless the brief says "review and fix"
- Final report: concise plain English — safe to proceed to Opus sign-off or not; list issues

## Constraints
- Do not nudge yourself toward approving
- Prefer evidence (file:line, failing scenarios)
EOF

grok --prompt-file "$PROMPT_FILE" \
  --cwd /path/to/repo \
  -m grok-4.5 \
  --always-approve \
  --output-format plain \
  > "$OUT"
cat "$OUT"
```

If findings need code changes → **`grok-subagent`** with the findings list (senior fix).

## After it finishes

Show the user the reviewer’s **exact response in full**. Do not rewrite it.

Then: fixes (Grok) → **Opus final sign-off**.

## Aliases

`/gpt-review` under Stack A maps here (**Grok 4.5** senior), not OpenAI GPT.
