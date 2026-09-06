# Skills layout

Not read by opencode — only `SKILL.md` files are discovered. This is for humans.

```
skill/
  <hub>/
    SKILL.md      discovered: frontmatter name+description, + a generated index
    <leaf>.md     invisible to discovery; the actual content
  CATALOG.tsv     generated: every leaf in one grep-able table
  README.md       this file
```

## Why hubs

opencode's skill listing is **not lazy**. Every discovered skill's name,
description, and absolute path go into the system prompt of every session, for
every agent holding the `skill` tool. Measured on this corpus: **~646 tokens
for 5 entries, ~130 each** — a ~300-char description dominates, the tags and
absolute path add ~35. Only the body is lazy (~24.8k tokens held back here).

A flat corpus of 100 skills would therefore cost ~7–9k tokens of a 64k window on
every request, and hand a 35B IQ3 model a 100-item menu. Hubs make the always-on
cost proportional to the number of *domains* instead of the number of documents:
4 hubs ≈ 500 tokens whether they hold 25 leaves or 250. Tightening a description
is the cheapest lever available — every char is spent on every request.

The mechanism is that every discovery glob requires the filename to be exactly
`SKILL.md`. Anything else is invisible to the loader but still on disk, so the
model loads a hub, reads its index, and opens the one leaf it needs.

## Constraints that are not stylistic

- **Leaves stay flat.** Skill dirs are auto-allowed through the
  external-directory boundary as `<dir>/*` — one level. A subdirectory can
  trigger a permission prompt on every read.
- **The generated index is authoritative.** The `skill` tool appends its own
  sibling-file list, but it caps at 10 and is explicitly sampled.
- **A skill with no `description` is dropped silently** and never offered.
- `name` must equal the directory name.

## Workflow

```bash
bin/skills-build            # rewrite hub indexes + CATALOG.tsv, and lint
bin/skills-build --check    # report drift, write nothing (exit 1 if stale)
bin/skills-import-awesome   # refresh opencode-ecosystem from awesome-opencode
```

Adding a leaf needs no restart — leaves are read at request time. Adding or
renaming a **hub** changes the system prompt, so restart opencode.

Leaf frontmatter:

```markdown
---
title: Short imperative phrase
when: comma-separated trigger conditions, not a summary
---
```

Both fields are required. `when` is the only thing the model sees when choosing
between leaves, so write it as symptoms and situations.

## Growing past this

At several hundred leaves, add a `skill-catalog` hub whose body says to grep
`CATALOG.tsv` — that is what the file is for. It keeps the always-on cost at one
entry for an unbounded corpus, at the price of a bash round-trip.
