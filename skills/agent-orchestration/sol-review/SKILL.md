---
name: sol-review
description: 'Adversarial code review by GPT-5.6 Sol (falls back to GPT-5.5) — the Stack A reviewer. Cross-vendor from Grok, which implements. Use after grok-subagent lands work and before Opus sign-off, or when the user says /sol-review. Reviews only; fixes go back to grok-subagent.'
disable-model-invocation: true
---

# GPT Sol — Stack A reviewer

Sol reviews. It does not implement. Findings go back to **Grok** (`grok-subagent`),
then to **Opus** for sign-off.

**Why a different vendor from the implementer:** a model asked to review its own
work approves it. Cross-vendor review is the only step in this pipeline that has
reliably found Criticals that green test suites missed.

## Model availability — check before assuming

Verified 2026-08-07:

| Model | Status |
|-------|--------|
| `gpt-5.6-terra` | available — **the fallback reviewer**, run at `high` effort |
| `gpt-5.6-luna` | available (junior lane) |
| `gpt-5.5` | available |
| `gpt-5.6-sol` | **NOT available on a ChatGPT-account Codex login** |

Sol returns:

```
The 'gpt-5.6-sol' model is not supported when using Codex with a ChatGPT account.
```

It needs API-key auth or a higher plan tier. **Probe first, fall back to Terra at
high reasoning effort:**

```bash
MODEL=gpt-5.6-sol
EFFORT=xhigh
if codex exec -m "$MODEL" --skip-git-repo-check -s read-only "reply OK" </dev/null 2>&1 \
     | grep -q "not supported"; then
  MODEL=gpt-5.6-terra
  EFFORT=high
fi
echo "reviewer: $MODEL (effort $EFFORT)"
```

Terra is still cross-vendor from Grok, so the point of the step survives the
fallback. Run it at `high` — review is the one stage where reasoning effort earns
its cost, and a cheap review is worse than none because it manufactures
confidence.

## Launch

```bash
REPO=/path/to/repo
PROMPT=$(mktemp /tmp/sol-review.XXXXXX)
OUT=$(mktemp /tmp/sol-out.XXXXXX)

cat > "$PROMPT" <<'BRIEF'
You are the senior reviewer. The implementer was Grok 4.5. Adversarial stance:
green CI is not a verdict.

## Under review
    git log --oneline <base>..HEAD
    git diff <base>..HEAD --stat

## Verified state (already re-run — do not re-verify)
(tests / typecheck / build results with real exit codes)

## Judge these specifically
(the decisions you are least sure about, named)

## What to do
- Separate Critical / Important / Minor
- Concrete fix per Critical and Important, with file:line
- Do NOT implement fixes
- Final verdict in plain English: safe to build on, or not

## Constraints
- Do not nudge yourself toward approving
- Evidence: file:line plus a concrete failing scenario
- If a finding can only be settled on a device or in a browser, say so rather
  than guessing
BRIEF

codex exec -m "$MODEL" -c model_reasoning_effort="$EFFORT" \
  --skip-git-repo-check -s read-only \
  -C "$REPO" "$(cat "$PROMPT")" </dev/null > "$OUT" 2>&1
cat "$OUT"
```

## Ask it the right questions

The review is only as good as what you point it at:

- **Name the decisions you are least confident in.** A reviewer told "challenge
  the chart library choice and the 1RM data source" finds more than one told
  "review this diff".
- **Ask for a ruling, not a confirmation**, on anything that will recur. "Where
  should the accent live on an observational screen?" produces a rule you can
  reuse; "is this accent right?" produces a yes.
- **Ask what could be quietly broken and still pass.** That single question has
  found a fully unit-tested subsystem the product never invoked.
- **Say what has never been verified.** If nothing has run on a device, tell it,
  and ask it to mark hardware-only findings as such.

## After it finishes

Show the report **verbatim**. Do not summarise away findings.

Then verify its Criticals yourself before acting — a reviewer can be wrong too,
and a reproduction takes minutes:

```bash
<minimal repro> > /tmp/repro.log 2>&1; echo "EXIT=$?"
```

Route confirmed findings to **`grok-subagent`** as a fix brief, in the reviewer's
own priority order. Then **`opus-review`** for sign-off.
