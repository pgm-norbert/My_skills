---
name: grok-review
description: 'Launch Grok 4.5 (usually via Grok CLI) for a deep, neutral senior-developer review. Use when the implementer was Opus 5, or the user says "/grok-review", "grok review". Stack A: opposite-family reviewer vs Opus implementer.'
---

# Grok Review (Stack A)

Launch **Grok 4.5** to review everything fully and carefully, as if it was a senior developer reviewing the work of a junior.

**When to use:** the implementer was **Opus 5** (or you need a second-family review after Opus edits).

## How to launch

Prefer headless Grok CLI (separate process — same mechanism as `grok-subagent`, review-only prompt):

```bash
OUT=$(mktemp /tmp/grok-review.XXXXXX)
PROMPT_FILE=$(mktemp /tmp/grok-review-prompt.XXXXXX)
cat > "$PROMPT_FILE" <<'EOF'
You are a senior engineer reviewing a junior's work. Be neutral. Do not redesign for taste.

## Scope
(paths / PR / branch / what changed)

## What to do
- Go deep: correctness, edge cases, security, missing tests, merge risk
- Do not implement fixes unless asked — report only
- Final report: concise plain English — merge-safe or not; critical/serious issues; how to fix

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

If the host can select **Grok 4.5** as a Task model, that is also fine — still keep the brief neutral.

## After it finishes

Show the user the reviewer’s **exact response in full**. Do not rewrite it.

## Aliases

`/gpt-review` under Stack A maps here (**Grok 4.5** reviewer), not OpenAI GPT.
