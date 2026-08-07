---
name: launch-subagent
description: 'Read BEFORE launching any subagent — Task tool, background agents, parallel agents, delegating work. Stack A model routing (Opus orchestrates, Grok 4.5 implements, GPT-5.6 Luna does bounded mechanical work, GPT-5.6 Sol reviews, Opus signs off) plus the brief-writing and verification rules that determine whether delegation works.'
disable-model-invocation: true
---

# Launching a subagent — Stack A routing

| Need | Route to |
|------|----------|
| Feature work, fixes, refactors, investigation | **`grok-subagent`** (Grok 4.5) |
| Bounded mechanical work, bulk edits, classification | **`luna-subagent`** (GPT-5.6 Luna) |
| Adversarial review | **`sol-review`** (GPT-5.6 Sol → GPT-5.5) |
| Commit / merge / push / PR | Sonnet 5 light ops |
| Final sign-off | **`opus-review`** (Opus 5, this session) |

Do not launch a subagent unless the user asked to delegate or parallelize, or
the task is clearly large independent work.

## The three rules that decide whether delegation pays

**1. Narrow the scope.** A brief covering many areas makes an agent build a tool
instead of doing the work. One task, named files, small turn budget.

**2. Give a reference, not a description.** Point at a commit — `git show <sha>
-- path` — and say "copy this exactly". Describing a pattern in prose produces a
second, different pattern.

**3. Verify the result yourself.** Agents report success on work that did not
build, was never committed, or was never rendered. Before believing any report:

```bash
git -C "$REPO" log --oneline <base>..HEAD   # did it commit?
git -C "$REPO" status --short               # or strand it uncommitted?
<verify> > /tmp/v.log 2>&1; echo "EXIT=$?"  # real exit code, never after a pipe
```

For UI work, look at it.

## Cross-vendor review is not optional

The implementer and the reviewer must be different vendors. Grok implements,
Sol reviews. Every Critical found in this pipeline has come from that step, not
from the test suite.

## Briefs go in a file, not a flag

Subagents start blind. Put goal, exact paths, constraints, invariants (each with
the failure it prevents), definition of done, and exact verify commands in the
prompt file.

Never print or commit an API key. Load via `source ~/.config/stack-a/env`.
