---
name: grok-subagent
description: 'Launch the NEWEST Grok (auto-discovered; grok-4.7 as of 2026-10-09) as the Stack A senior engineer — the default implementer. Use when a non-Grok host (Opus or Sonnet) delegates a self-contained coding task, and when fixing findings from sol-review. If this session is already Grok TUI, do not launch this skill — implement here and read stack-a-grok-host.'
---

# Grok (newest) — senior engineer (Stack A default implementer)

Grok is Stack A's **default coder**. It implements from a complete brief and it
fixes what the reviewer finds.

## If you are already the Grok host

Stop. Do **not** launch this skill. Implement in the current session.
Read **`stack-a-grok-host`**. Spawning another Grok from Grok TUI is same-vendor
and a wasted round-trip.

This skill is for a **non-Grok host** (Opus / Sonnet in Claude Code) to launch
Grok as a separate process.

| Role | Model | Skill |
|------|-------|-------|
| Orchestrator | Opus 5 or Sonnet 5 (host) | — |
| **Senior engineer** | **Newest Grok** (grok-4.7 as of 2026-10-09) | **this skill** |
| Junior worker | Newest GPT Luna (gpt-5.6-luna) | `luna-subagent` |
| Reviewer | Newest GPT flagship (GPT-6-Astra) → older flagships | `sol-review` |
| Sign-off | Opus 5 (Normal) / host (Saver or Grok-host) | `opus-review` |

**The reviewer is deliberately a different vendor.** Never ask Grok to review its
own work — in practice a model passes its own output every time.

## Preflight

```bash
command -v grok >/dev/null || echo "MISSING: grok CLI (https://x.ai/cli)"
```

Not authenticated → tell the user to log in via `grok` once. Never read or print auth tokens.

## Launch

Write the full brief to a file, then run from the repo root:

```bash
REPO=/path/to/repo
eval "$(~/.config/stack-a/stack-a-models)"   # GROK_IMPLEMENTER = newest grok-X.Y (never pin)
PROMPT=$(mktemp /tmp/grok-brief.XXXXXX)

cat > "$PROMPT" <<'BRIEF'
# Task
(goal, exact files, constraints, definition of done)

# Context
(what already exists, what must not change, invariants that cost something to learn)

# Verify
(exact commands — and how to capture their REAL exit code)

# Return
- what you changed and what you deliberately left
- verification output with real exit codes
- every deviation from the brief, with reasoning
- anything you could not verify
BRIEF

grok --prompt-file "$PROMPT" \
  --cwd "$REPO" \
  -m "$GROK_IMPLEMENTER" \
  --always-approve \
  --output-format plain
```

Run it in the background for anything non-trivial and poll, rather than blocking.

## Writing the brief — what actually determines success

Subagents start blind. Every one of these has caused a real failure when omitted:

1. **Name the exact files.** A brief listing seven areas of work invites the agent
   to build a codemod instead of doing the work. Narrow scope beats broad scope.
2. **Give a reference, not a description.** Point at a commit
   (`git show <sha> -- path/to/file`) and say "copy this approach exactly".
   Prose descriptions of a pattern produce a second, different pattern.
3. **State the invariants that cost something to learn**, each with the failure it
   prevents. "Weight is integer hundredths of a kg" is forgettable; "never
   re-derive a focused input from stored value — that logged 102.5 kg as 1025 kg"
   is not.
4. **Give mechanical pass/fail checks**, not "make sure it works". A command that
   must return 0 cannot be self-reported as passing.
5. **Forbid the tempting shortcut by name.** If there is an easy fix that would
   hide the symptom rather than solve it, say so explicitly.

## After it finishes — verify before believing

Agents have reported "verified" on work that did not build, was never committed,
and was never rendered. Check, in this order:

```bash
git -C "$REPO" log --oneline <base>..HEAD    # did it actually commit?
git -C "$REPO" status --short                # or is it stranded uncommitted?
```

Then re-run the verification yourself and capture the **real** exit code — never
read `$?` after a pipe, which reports the last command in the pipeline:

```bash
<verify-command> > /tmp/v.log 2>&1; echo "EXIT=$?"
```

For UI work, look at it. A DOM or screenshot check catches in one command what a
report will not tell you.

## Then

Findings → **`sol-review`** (cross-vendor). Fixes → back here (or, on a Grok host,
fix in-session). Sign-off → **Opus** in Normal mode; the host verifies in Saver
or Grok-host mode.

## Credentials

Load via `source ~/.config/stack-a/env`. Never print or commit a key.
