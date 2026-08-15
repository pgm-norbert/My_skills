---
name: launch-subagent
description: 'Read BEFORE launching any subagent — Task tool, background agents, parallel agents, delegating work. Stack A routing (Opus/Sonnet/Grok-host orchestrates, Grok implements, Luna does mechanical work, Sol/Terra reviews) plus the brief-writing and verification rules that determine whether delegation works.'
disable-model-invocation: true
---

# Launching a subagent — Stack A routing

First: **which host is this?**

| Host | What to do |
|------|------------|
| **Grok TUI** (Opus at session limit, or user opened Grok) | Read **`stack-a-grok-host`**. Implement here. Do not launch `grok-subagent`. |
| **Claude Code / Opus** (Normal) | Plan here. Launch `grok-subagent` for coding. |
| **Claude Code / Sonnet** (Opus-Saver, ~70% Opus usage) | Same pipeline. Sonnet orchestrates; Grok still implements. |

| Need | Route to |
|------|----------|
| Feature work, fixes, refactors (non-Grok host) | **`grok-subagent`** (Grok 4.5) |
| Same, but this session is already Grok | **this session** (`stack-a-grok-host`) |
| Bounded mechanical work, bulk edits, classification | **`luna-subagent`** (GPT-5.6 Luna) |
| Adversarial review | **`sol-review`** (GPT-5.6 Sol → Terra /high) |
| Commit / merge / push / PR | Sonnet 5 light ops (or this host, if already Grok) |
| Final sign-off | **`opus-review`** — or the Grok host if Opus is maxed |

Do not launch a subagent unless the user asked to delegate or parallelize, or
the task is clearly large independent work — **except** a non-Grok host must
still send coding to Grok rather than implementing in Opus/Sonnet.

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
Sol/Terra reviews. Every Critical found in this pipeline has come from that step,
not from the test suite.

## Briefs go in a file, not a flag

Subagents start blind. Put goal, exact paths, constraints, invariants (each with
the failure it prevents), definition of done, and exact verify commands in the
prompt file.

Never print or commit an API key. Load via `source ~/.config/stack-a/env`.
