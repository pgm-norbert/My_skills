---
name: grok-review
description: 'MOVED. Grok is the Stack A implementer, not the reviewer. Adversarial review is sol-review (newest GPT flagship (GPT-6-Astra) → older flagships) — deliberately a different vendor. Use grok-subagent to implement and fix. If this session is already Grok TUI, use stack-a-grok-host.'
disable-model-invocation: true
---

# grok-review has moved (2026-08-07)

the newest Grok (4.7) is the **implementer**, so it can no longer be the reviewer — a
model asked to review its own work approves it.

- Review → **`sol-review`**
- Implement and fix (from a non-Grok host) → **`grok-subagent`**
- Implement and fix (this session is Grok TUI) → **`stack-a-grok-host`**
- Sign-off → **`opus-review`** (or the Grok host, if Opus is at session limit)

If you reached this skill from an old brief or habit, use `sol-review`.
