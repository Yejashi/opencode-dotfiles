---
title: The MEMORY.md protocol
when: finishing significant work, recording a decision or blocker, or looking for prior context on this machine
---

# Memory protocol

Two layers, both under `/home/yejashi`:

- `MEMORY.md` — high-level state. Current state, what is done, what is next,
  environment facts, and a "recent notes" index.
- `memory/<YYYY-MM-DD>-<topic>.md` — one note per piece of work, with the
  detail and the reasoning.

Nothing loads these automatically — like the NixOS setup, there is no global
`~/.config/opencode/AGENTS.md`. Read them when a task touches this machine.

## Reading

At session start, read `MEMORY.md`. Read the specific note under `memory/` only
when the task touches it — the index lines in `MEMORY.md` say what each covers.

## Writing

After significant work, a decision, a blocker, or a state change:

1. Update the relevant `memory/<date>-<topic>.md`, or create one.
2. Update `MEMORY.md` — the current-state bullets **and** the recent-notes index
   line if the note is new.

Conventions that matter:

- Concise and factual. Record the *why*, and the measurement if there was one —
  the useful notes here are the ones with numbers.
- Convert relative dates to absolute.
- **Never** write secrets, tokens, passwords, or API keys.
- Record what was *not* obvious. Things recoverable from the code or from
  `git log` do not belong here.

## Overlap with skills

These are different tools. A memory note is a dated record of what happened; a
skill leaf is durable instruction for how to do something. When a note's
findings stop being news and start being procedure, promote them into a skill
leaf and leave the note as the record.
