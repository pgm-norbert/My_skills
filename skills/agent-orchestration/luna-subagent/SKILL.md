---
name: luna-subagent
description: 'Launch GPT-5.6 Luna as the Stack A junior worker for bounded mechanical work — bulk edits across many files, short single-file changes, classification. NOT the default implementer; feature coding goes to grok-subagent. Keep its scope narrow.'
disable-model-invocation: true
---

# GPT-5.6 Luna — junior worker

Luna handles **bounded mechanical work**. It is not the default implementer —
that is Grok (`grok-subagent`).

Use it for:
- bulk find-and-replace across many files where the mapping is already decided
- short single-file edits
- classification, labelling, high-volume cheap inference

Do **not** use it for: feature work, anything requiring design judgement, or
anything where "which option is right here" is part of the task.

## Preflight

`gpt-5.6-luna` was verified available on the ChatGPT-account Codex login
(2026-08-07). Confirm before a long run:

```bash
codex exec -m gpt-5.6-luna --skip-git-repo-check -s read-only "reply OK" </dev/null
```

## Launch

```bash
REPO=/path/to/repo
codex exec -m gpt-5.6-luna \
  --skip-git-repo-check \
  -C "$REPO" \
  "$(cat /tmp/luna-brief.md)" </dev/null
```

## Scope discipline — the failure mode to design against

A junior given a broad brief **builds a tool instead of doing the work**. Given
seven areas and a large turn budget, it will spend the whole budget writing a
codemod and ship nothing. This has happened.

So:

1. **One task. Named files.** Not "restyle the app" — "edit these four files".
2. **State the mapping explicitly.** If it is a find-and-replace, give the table.
   Do not make it infer the rule.
3. **Forbid tooling by name:** "Do not write a codemod, a script, or a shared
   utility. Edit these files by hand."
4. **Small turn budget.** A large one is an invitation to over-engineer.
5. **Mechanical pass/fail check** it must run before reporting done.

## Verify before believing

Bulk edits fail silently and at scale. Always check structural integrity, not
just the intended change:

```bash
# example: JSX attributes glued together by an over-eager whitespace rule
grep -rcoE '"[a-zA-Z-]+=' <touched-files> | awk -F: '{s+=$2} END{print s+0}'   # want 0
```

Then a typecheck, then read a diff hunk with your own eyes. A transform that
"worked" on 9 files has corrupted 90 attributes across them before.

## Then

Anything Luna touched still goes through **`sol-review`** and **Opus sign-off**
like any other change.
