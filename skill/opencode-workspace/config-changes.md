---
title: Change opencode.json safely
when: editing OpenCode config, a change appears to have no effect, or OpenCode refuses to start after an edit
---

# Changing config

## It is not hot-reloaded

Config is read once at startup. After editing `opencode.json`, an agent prompt,
a hub `SKILL.md`, or the `lsp` key, **quit and restart OpenCode**. A running
session keeps the config it loaded. This is the single most common reason a
change "did nothing".

Leaf `.md` files are the exception — they are read at request time, so editing
one takes effect immediately.

## Before editing

OpenCode validates strictly and **refuses to start** on an invalid config, so a
mistake costs a session. The convention here is a timestamped backup beside the
file:

```bash
cp ~/.config/opencode/opencode.json \
   ~/.config/opencode/opencode.json.bak-$(date +%Y%m%d-%H%M%S)
```

Validate the shape against `https://opencode.ai/config.json` rather than
guessing; unknown top-level keys are rejected outright. Keep the `$schema` line.

## After editing

```bash
python3 -m json.tool ~/.config/opencode/opencode.json >/dev/null   # syntax
opencode debug config                                              # merged result
opencode debug agent <name>                                        # one agent
opencode debug skill                                               # skills found
```

`opencode debug config` failing *is* the check — it loads config the same way a
real session does.

## When it will not start

```bash
OPENCODE_DISABLE_PROJECT_CONFIG=1 opencode   # skip the project config
OPENCODE_CONFIG=/path/to/file.json           # load an explicit config
OPENCODE_PURE=1 opencode                     # skip external plugins
```

These let you get a session open to fix the broken file from inside.

## Prefer files over inline

Agent prompts belong in `prompts/<name>.md` referenced as
`{file:./prompts/<name>.md}`, and skills in `skill/<name>/SKILL.md` — not
inlined into `opencode.json`.
