---
name: opus-review
description: 'Launch an Opus 5 subagent (or host Task on Opus 5) for a deep, neutral senior-developer review. Use when the implementer was Grok 4.5 / DeepSeek / Qwen, or the user says "/opus-review", "opus review". Stack A: opposite-family reviewer vs Grok implementer.'
---

# Opus Review (Stack A)

Launch an **Opus 5** reviewer to review everything fully and carefully, as if it was a senior developer reviewing the work of a junior.

**When to use:** the hard executor was **Grok 4.5** (or a cheap implementer). Opposite-family review is the point.

## How to launch

Prefer, in order:

1. **Host Task / Agent tool** with model **Opus 5** (same product session family — fine for review when implementer was Grok CLI).
2. If the host is already Opus 5 and cannot spawn a second Opus cleanly, open a **fresh Opus session** (new Claude Code chat / pane) with only the review brief — not the implementation chat history that might bias it.
3. Do **not** use Grok as the reviewer when Grok was the implementer.

## Brief for the reviewer

Give necessary context, but stay neutral and unbiased. Do not nudge toward any one specific solution. The goal is great work — be objective.

Tell it what to review, but don’t be overly specific — let it find its own bugs and shortcomings. Tell it to work extremely hard, go deep, and surface any critical or serious issues.

Ask for a concise plain-English report: safe to merge / not, with serious issues and how to fix them.

## After it finishes

Show the user the reviewer’s **exact response in full**. Do not rewrite it. Do not soften it.

## Aliases

`/fable-review` and older “Fable Max reviewer” language map to this skill under Stack A (**Opus 5**).
