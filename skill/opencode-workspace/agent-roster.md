---
title: The agent roster and when to use each
when: choosing between build, orchestrate-local, orchestrate-frontier, raw, or a worker (explore, implementer, operator, tester); editing an agent definition; or deciding whether delegation is worth it
---

# Agent roster

Defined in `~/.config/opencode/opencode.json` under `agent`, prompts in
`~/.config/opencode/prompts/`. Same roster as the NixOS machine;
`~/.config/opencode/ORCHESTRATION.md` is the overview.

| Agent | Mode | Model | Tools | Use for |
| --- | --- | --- | --- | --- |
| `build` | primary (built-in) | picker | everything | Ordinary work |
| `orchestrate-local` | primary | local Qwen | glob, grep, webfetch, task, todowrite — no read/edit/bash | Large multi-file work, fully local |
| `orchestrate-frontier` | primary | `openai/gpt-5.6-terra` | same as above | Same, with a frontier planner; needs `OPENAI_API_KEY` |
| `raw` | primary | local heretic | none | Non-coding chat and creative writing |
| `explore` | subagent | local Qwen | read, glob, grep | One narrow codebase question |
| `implementer` | subagent | local Qwen | read, write, edit, patch | One already-decided edit to known files |
| `operator` | subagent | local Qwen | bash only | Exact state-changing commands: git add/commit/push, installs |
| `tester` | subagent | local Qwen | bash only, never edits | Read-only git inspection, diagnostics, builds, tests |

Both orchestrators may delegate **only** to the four workers
(`permission.task` denies everything else). Every worker sets
`doom_loop: deny`, and the local provider sets `parallel_tool_calls: false`, so
each command result is seen before the next action is chosen.

## When delegation actually pays

Measured on this setup with the earlier single `orchestrate` agent, both arms
verified correct:

| Task size | `build` peak context | `orchestrate` peak | Wall clock |
| --- | --- | --- | --- |
| 3 tasks, ~20 lines | 10,669 tok (16.3%) | 8,541 tok (13.0%) | 2.43x slower |
| 12 functions, ~240 lines | 19,125 tok (29.2%) | 12,083 tok (18.4%) | 1.90x slower |

The benefit scales with task size; the ~1.9x wall-clock cost does not go away.
Use an orchestrator when the window is the binding constraint, `build`
otherwise. `orchestrate-frontier` changes the trade: the planner has a large
window and the frontier API sees only the conversation and compact worker
reports, never the workers' tool loops.

## Why the orchestrators have no read tool

That is deliberate — it forces delegation instead of quietly doing the work
itself. The consequence for skills: an orchestrator can load a hub SKILL.md but
cannot open a leaf, and `implementer`, `operator`, and `tester` have the skill
tool disabled. Name the leaf's absolute path in the task and let `explore` or
`implementer` (both have `read`) open it.

## The one lever that works

Per-agent sampling and reasoning settings do not reach the local provider — see
the local-model limits topic. The only working lever on local orchestrator
quality is the prompt. Rewriting `prompts/orchestrate.md` to demand a GOAL /
UNKNOWNS / UNITS / RISK plan before the first delegation cut peak context
18.4% → 15.1% and orchestrator calls 9 → 7 on the same task.
