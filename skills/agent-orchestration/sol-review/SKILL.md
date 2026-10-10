---
name: sol-review
description: 'Adversarial code review by the NEWEST OpenAI GPT flagship (auto-discovered; GPT-6-Astra as of 2026-10-09, then older flagships as fallback) — the Stack A reviewer. Cross-vendor from Grok, which implements. Use after grok-subagent lands work and before Opus sign-off, or when the user says /sol-review or /gpt-review. Reviews only; fixes go back to grok-subagent.'
disable-model-invocation: true
---

# GPT review — Stack A reviewer (newest GPT flagship)

The newest GPT flagship reviews. It does not implement. (The skill keeps the name
`sol-review` for compatibility — the model is whatever is newest, not necessarily Sol.) Findings go back to **Grok** (`grok-subagent`),
then to **Opus** for sign-off.

**Why a different vendor from the implementer:** a model asked to review its own
work approves it. Cross-vendor review is the only step in this pipeline that has
reliably found Criticals that green test suites missed.

## Model selection — always the newest, discovered at run time

Never hardcode a reviewer model. OpenAI ships new flagships often; a pinned name
goes stale. Discover the chain from the Codex CLI's own model list:

```bash
eval "$(~/.config/stack-a/stack-a-models)"   # sets GPT_REVIEWERS / GPT_REVIEWER
echo "reviewer chain: $GPT_REVIEWERS"         # e.g. gpt-6-astra gpt-5.6-sol gpt-5.6-terra gpt-5.5
```

As of 2026-10-09 that resolves to **`gpt-6-astra`** (GPT-6-Astra) first, then
Sol → Terra → 5.5. Script source: `~/Documents/My_skills/bin/stack-a-models`
(reads `~/.codex/models_cache.json`; no network, no tokens).

**Probe, then fall down the chain.** A listed model can be plan-gated, and the
whole ChatGPT account can hit its usage limit:

```bash
MODEL=""
for attempt in 1 2; do                      # right at a usage reset the first probe can fail without the limit message
[ -n "$MODEL" ] && break
[ "$attempt" = 2 ] && sleep 60
for m in $GPT_REVIEWERS; do
  out=$(codex exec -m "$m" --skip-git-repo-check -s read-only "reply with exactly: OK" </dev/null 2>&1)
  if echo "$out" | grep -q "usage limit"; then
    echo "ChatGPT usage limit hit: $(echo "$out" | grep -o 'try again at [^.]*' | head -1)"
    break                                   # every GPT on this account is out — stop probing
  fi
  # success = the model actually answered; anything else (plan-gated, transient error) → next
  if echo "$out" | grep -qx "OK"; then MODEL=$m; break; fi
done
done
EFFORT=xhigh   # flagship; use high for an older fallback
echo "reviewer: ${MODEL:-NONE}"
```

**If `MODEL` is empty because of the usage limit:** do not merge high-risk work on
Grok's word alone. Either wait for the reset time printed above, or — only when
the user agrees — run the review with a Claude model that is not the orchestrator
(e.g. `fable-review`). That is still cross-vendor from the Grok implementer.

Run at `xhigh` for the flagship. Review is the one stage where reasoning effort
earns its cost; a cheap review manufactures confidence.

## Launch

```bash
REPO=/path/to/repo
PROMPT=$(mktemp /tmp/sol-review.XXXXXX)
OUT=$(mktemp /tmp/sol-out.XXXXXX)

cat > "$PROMPT" <<'BRIEF'
You are the senior reviewer. The implementer was Grok. Adversarial stance:
green CI is not a verdict. Review it yourself — do NOT spawn sub-agents or
load orchestration skills (it burns the shared usage limit).

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
