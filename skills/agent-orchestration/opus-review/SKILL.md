---
name: opus-review
description: 'Opus 5 final sign-off (Stack A). Use after DeepSeek junior + Grok senior review/fix, or when the user says "/opus-review", "opus review", "final sign-off". Last gate before claiming done or merge-ready.'
---

# Opus Review — Final Sign-off (Stack A)

Launch **Opus 5** (or use the host Opus orchestrator session) for the **last gate** before claiming work is done or merge-ready.

**When to use:**

- After **DeepSeek junior** implemented and **Grok senior** reviewed (and fixed if needed).
- User says `/opus-review`, “opus review”, “sign off”, “final review”.
- Anytime the pipeline completed and someone is about to say “done”.

Stack A pipeline reminder:

`Opus plan → DeepSeek implement → Grok senior review/fix → **Opus sign-off**`

## How to launch

Prefer, in order:

1. **Host Opus 5 session** (orchestrator) performing sign-off with a clean checklist if it did not write the code.
2. **Host Task / Agent tool** with model **Opus 5** and a sign-off-only brief.
3. **Fresh Opus session** (new Claude Code chat / pane) with only the sign-off brief — useful if the implementation chat is huge or biased.

## Brief for sign-off

Give context, stay objective:

```markdown
You are doing FINAL SIGN-OFF, not a redesign.

## What shipped
(summary from junior + senior)

## Diff / range
(base..head or PR)

## Evidence
(test commands + results)

## Prior senior review
(Grok findings + what was fixed)

## Decide
- Sign off: yes / no
- If no: only blocking issues + how to fix
- Do not re-litigate pure taste if senior already approved
```

Ask for a concise plain-English report: **signed off or not**, with any remaining blockers.

## After it finishes

Show the user the sign-off report **in full**. Do not soften blockers.

## Aliases

`/fable-review` maps here under Stack A (**Opus 5** final sign-off).
