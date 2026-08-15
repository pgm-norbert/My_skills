---
name: stack-a-grok-host
description: 'Run Stack A when Grok is the host because Opus hit its session limit or the user opened Grok TUI. Triggers: "we are in Grok", "Opus is maxed", "session limit", "carry on without Opus", "Grok TUI host". Differentiator: implement in this session — do not spawn grok-subagent.'
---

# Stack A — Grok is the host

Opus is unavailable (session limit) or the user opened this Grok TUI on purpose.
You are the orchestrator **and** the implementer.

## Seats

| Seat | Who |
|------|-----|
| Orchestrator + implementer | **This Grok session** |
| Reviewer | **`sol-review`** (GPT-5.6 Sol → Terra /high) — different vendor, required |
| Mechanical grunt | **`luna-subagent`** — bulk find/replace only |
| Opus | Out. Do not wait. Do not spawn. High-risk sign-off is deferred. |

Cross-vendor review still holds: Grok writes, a GPT reviews. Same-vendor review is not a review.

## Pipeline

1. If a `=== REMEMBER ===` / `=== HANDOFF ===` block is present, open with one line: `Resuming <project>: <state> · Next: <item>`.
2. Plan briefly, then **implement here**. Do **not** launch `grok-subagent` — that is another Grok, same vendor, wasted round-trip.
3. Re-run tests / typecheck / read the diff yourself. Capture real exit codes (`cmd > /tmp/v.log 2>&1; echo EXIT=$?`).
4. Run **`sol-review`**. Show the report verbatim. Fix findings in this session.
5. Sign off yourself with evidence. State what you could not verify.
6. Commit only if the user asked.

## Do not

- `/model sonnet` — that is Claude Code. You are Grok TUI.
- Call Opus for routine sign-off or block the task waiting for it.
- Review your own diff and call it cross-vendor review.
- Default coding to Luna, Sonnet, or local models.
- Follow `deepseek-subagent` — DeepSeek is retired.

## Review

Follow `sol-review`. Probe Sol first; fall back to Terra at `high`.

```bash
MODEL=gpt-5.6-sol
EFFORT=xhigh
if codex exec -m "$MODEL" --skip-git-repo-check -s read-only "reply OK" </dev/null 2>&1 \
     | grep -q "not supported"; then
  MODEL=gpt-5.6-terra
  EFFORT=high
fi
```

## When Opus comes back

Normal mode: Opus host plans, `grok-subagent` implements, `sol-review` reviews, Opus signs off.
