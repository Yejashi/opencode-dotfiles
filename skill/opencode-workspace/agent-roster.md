---
title: The agent roster and when to use each
when: choosing between build, orchestrate, raw, implementer, or tester; editing an agent definition; or deciding whether delegation is worth it
---

# Agent roster

Defined in `~/.config/opencode/opencode.json` under `agent`, prompts in
`~/.config/opencode/prompts/`.

| Agent | Mode | Tools | Use for |
| --- | --- | --- | --- |
| `build` | primary (built-in) | everything | Ordinary work. **The default; keep it that way.** |
| `orchestrate` | primary | glob, grep, task, todowrite — no read/edit/bash | Large multi-file work that would otherwise fill the window |
| `raw` | primary | none | Non-coding chat and creative writing |
| `implementer` | subagent | read, write, edit, patch, glob, grep, bash | One already-decided change |
| `tester` | subagent | bash, read, glob, grep — never edits | Running commands and reporting pass/fail |

## When delegation actually pays

Measured on this setup, both arms verified correct:

| Task size | `build` peak context | `orchestrate` peak | Wall clock |
| --- | --- | --- | --- |
| 3 tasks, ~20 lines | 10,669 tok (16.3%) | 8,541 tok (13.0%) | 2.43x slower |
| 12 functions, ~240 lines | 19,125 tok (29.2%) | 12,083 tok (18.4%) | 1.90x slower |

The benefit scales with task size; the ~1.9x wall-clock cost does not go away.
Peak-context saving went 20% → 37% as the task grew. Use `orchestrate` when the
window is the binding constraint, `build` otherwise. `build` already delegates
to `explore` on its own, so the baseline is not "no delegation".

## Why orchestrate has no read tool

That is deliberate — it forces delegation instead of quietly doing the work
itself. The consequence for skills: `orchestrate` can load a hub SKILL.md but
cannot open a leaf. It should read the hub to decide *which* skill to name in
the task prompt, and let `implementer` or `tester` read the leaf.

## The one lever that works

Per-agent sampling and reasoning settings do not reach this provider — see the
local-model limits topic. The only working lever on orchestrator quality is the
prompt. Rewriting `prompts/orchestrate.md` to demand a GOAL / UNKNOWNS / UNITS /
RISK plan before the first delegation cut peak context 18.4% → 15.1% and
orchestrator calls 9 → 7 on the same task.
