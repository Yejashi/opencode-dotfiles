---
title: Which model an agent actually runs on
when: an agent runs on an unexpected model, you are adding or removing an agent's model pin, or you want subagents to follow the picker
---

# Model resolution

Verified against the opencode 1.18.25 binary.

## Primary agents

```
request model  ??  agent.model  ??  session model
```

The model you choose in the picker is sent with the request and wins. An agent
with **no `model` key runs on whatever is currently selected** — that is the
whole mechanism, there is no hidden default.

## Subagents (spawned via the task tool)

```
agent.model  ??  { model of the parent assistant message }
```

An unpinned subagent inherits the model of whatever spawned it. So
`orchestrate` on the base model produces `implementer` runs on the base model,
automatically.

## Variant inheritance

The task tool passes `variant: agent.model ? undefined : parentVariant`. A
subagent with **no** model pin inherits the parent's variant; pinning a model
resets the variant to the model default. One more reason to leave subagents
unpinned.

## Current state here

`orchestrate`, `implementer`, and `tester` are unpinned and follow the picker.
`raw` is pinned to `local/qwen3.6-35b-a3b-heretic` on purpose.

That pin has a cost worth remembering: only one ~13 GiB model fits in 16 GiB of
VRAM, so switching to `raw` evicts whatever is loaded and pays a ~14 s swap.

## Checking it

```bash
opencode debug agent orchestrate     # resolved config for one agent
opencode debug config                # the whole merged config
models_llm                           # which model the router has resident
```
