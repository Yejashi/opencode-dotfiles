---
title: Add or edit a skill in this workspace
when: writing a new skill, adding a topic to an existing one, or the hub index looks out of date
---

# Authoring skills here

Layout under `~/.config/opencode/skill/`:

```
<hub>/
  SKILL.md      discovered by opencode; frontmatter + a generated index
  <leaf>.md     invisible to opencode; the actual content
CATALOG.tsv     generated; every leaf in one grep-able table
```

## Adding a topic to an existing hub

1. Create `<hub>/<topic>.md` with this frontmatter:

```markdown
---
title: Short imperative phrase naming the topic
when: the situations that should send a reader here, comma separated
---

# Content
```

2. Run `~/.config/opencode/bin/skills-build`.
3. Restart OpenCode only if you added or changed a **hub** — leaf edits need no
   restart, because leaves are read at request time, not load time.

`title` and `when` are both required; the build fails loudly without them.
Write `when` as trigger conditions, not a summary — it is the only thing a
model sees when choosing between twenty leaves.

## Adding a hub

Hubs are expensive: each one costs ~130 tokens in *every* system prompt
forever, most of it the description. Add one only for a domain with several topics. Create
`<hub>/SKILL.md`:

```markdown
---
name: <must equal the directory name>
description: Use when … front-load the literal words and filenames someone would say.
---

Two or three sentences of orientation.

<!-- BEGIN GENERATED INDEX -->
<!-- END GENERATED INDEX -->
```

Then `skills-build`, then decide which agents should see it — see the
permissions topic — then restart OpenCode.

## The build script

```bash
~/.config/opencode/bin/skills-build            # rewrite indexes + CATALOG.tsv
~/.config/opencode/bin/skills-build --check    # report drift, write nothing
```

It also lints: missing `description` (opencode would drop the skill silently),
`name` not matching the directory, duplicate names, empty hubs, nested
subdirectories, and over-long descriptions.

## Rules that come from the loader, not taste

- Filename must be exactly `SKILL.md` — that is what makes leaves invisible.
- Leaves stay **flat** beside their SKILL.md; no subdirectories.
- The generated index is authoritative; the tool's own sampled file list caps
  at 10 entries.
