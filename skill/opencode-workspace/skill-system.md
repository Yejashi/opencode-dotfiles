---
title: How OpenCode finds and loads skills
when: a skill is not offered to the model, you are deciding where to put one, or you are worried about skills bloating the system prompt
---

# The skill system

Verified against the opencode 1.18.25 binary.

## Where it scans

| Location | Glob |
| --- | --- |
| `~/.config/opencode/{skill,skills}/**/SKILL.md` | the one used here |
| `<project>/.opencode/{skill,skills}/**/SKILL.md` | per project |
| `~/.claude/skills/**/SKILL.md`, `~/.agents/skills/**/SKILL.md` | Claude Code compat |
| `.claude/skills/`, `.agents/skills/` walking up from cwd | project compat |
| `skills.paths` in `opencode.json` | `**/SKILL.md` |
| `skills.urls` | remote; fetches `index.json` first |

Every glob requires the filename to be **exactly `SKILL.md`**. That is the
lever the hub-and-leaf layout depends on: any other `.md` is invisible to
discovery while still sitting on disk, readable on demand.

Env killswitches: `OPENCODE_DISABLE_EXTERNAL_SKILLS=1`,
`OPENCODE_DISABLE_CLAUDE_CODE_SKILLS=1`.

## What it costs

**The skill listing is not lazy.** Every discovered skill's name, description,
*and absolute location path* go into the system prompt of every session for
every agent that has the `skill` tool. Measured here: **~130 tokens per entry**
with a ~300-char description. Only the body is lazy, loaded when the model calls
the `skill` tool by name.

At 128k context that means 100 flat skills would burn ~13k tokens — a tenth of
the window — on every request, and hand a 35B IQ4_XS model a 100-item menu.
Hence hubs.

A skill with **no `description` is silently dropped** and never offered.

## Two behaviours worth knowing

- The `skill` tool appends a sibling-file list, but it is `limit: 10` and
  captioned "file list is sampled". Never rely on it — the hub body carries the
  authoritative index.
- Skill directories are auto-allowed through the external-directory boundary,
  but the generated rule is `<dir>/*`, **one level only**. Leaves must sit
  directly beside their `SKILL.md`, or reads can trigger a permission prompt.

## Inspecting

```bash
opencode debug skill        # exactly what the loader found, with full content
```
