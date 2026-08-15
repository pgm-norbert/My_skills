---
name: opus-review
description: 'Opus 5 final sign-off (Stack A). Use after Grok implemented and Sol/Terra reviewed and findings were fixed, or when the user says /opus-review. Last gate before claiming done. If Opus is at session limit, the Grok host verifies instead — read stack-a-grok-host, do not block.'
disable-model-invocation: true
---

# Opus 5 — final sign-off

The last gate. Runs in the Opus host session; nothing is delegated.

`plan → Grok implement → Sol/Terra review → Grok fix → **sign-off**`

## If Opus is at session limit

Do **not** block. The Grok host verifies independently (see `stack-a-grok-host`).
Defer only high-risk sign-off (migrations, security, irreversible/prod ops) until
Opus is back. Say that plainly in the handoff.

## When this skill runs (Opus host)

Use it after Grok has implemented and Sol's findings have been fixed, on anything
non-trivial.

## Sign-off is verification, not summary

Do not sign off on an agent's report. Every claim gets checked here, because
agents have reported "verified" on work that did not build, was never committed,
and was never rendered on screen.

**1. Did the work actually land?**

```bash
git log --oneline <base>..HEAD    # commits exist?
git status --short                # or is it stranded uncommitted?
```

**2. Re-run every gate yourself, capturing real exit codes.** Never read `$?`
after a pipe — it reports the last command in the pipeline, and that has masked
a completely broken build.

```bash
<test>      > /tmp/t.log  2>&1; echo "TEST=$?"
<typecheck> > /tmp/tc.log 2>&1; echo "TYPECHECK=$?"
<build>     > /tmp/b.log  2>&1; echo "BUILD=$?"
```

**3. Reproduce the fix independently.** If a Critical was fixed, run the failing
scenario yourself and watch it pass. If a test was added to cover wiring, delete
the wiring and confirm that test fails — coverage that survives deletion is
decorative.

**4. Look at it.** For anything with a UI, open it. Read the DOM, take a
screenshot. Tests confirm the paths someone thought to write; they do not tell
you the product renders.

## Say what is not done

A sign-off that hides gaps is worse than a refusal. State plainly:
- what was fixed and verified
- what was deliberately deferred, and why
- what could not be verified here — device-only behaviour, real data, load

## Refuse when it is not ready

"Not signed off" is a valid outcome. Say what blocks it and what would clear it.
Green gates are necessary, not sufficient — every Critical in this pipeline was
found by adversarial review or by looking, never by a passing suite.
