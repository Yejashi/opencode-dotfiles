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

An unpinned subagent inherits the model of whatever spawned it. Under
`orchestrate-frontier` that would be GPT-5.6 Terra — every worker's tool loop
would go to the paid API. That is why all four workers here are pinned.

## Variant inheritance

The task tool passes `variant: agent.model ? undefined : parentVariant`. A
subagent with **no** model pin inherits the parent's variant; pinning a model
resets the variant to the model default.

## Current state here

Every agent is pinned:

| Agent | Model |
| --- | --- |
| `orchestrate-local` | `local/qwen3.6-35b-a3b` |
| `orchestrate-frontier` | `openai/gpt-5.6-terra` (reasoningEffort medium) |
| `explore`, `implementer`, `operator`, `tester` | `local/qwen3.6-35b-a3b` |
| `compaction`, `summary` | `local/qwen3.6-35b-a3b` |
| `title` | `local-title/qwen3.6-35b-a3b` — same server, 256 output tokens |
| `raw` | `local/qwen3.6-35b-a3b-heretic`, on purpose |

`small_model` is also local, so no metadata work reaches the frontier API.

The `raw` pin has a cost worth remembering: only one model is resident at a
time (each is capped at 13 GiB of VRAM), so switching to `raw` evicts whatever
is loaded and pays a ~15 s swap.

## Checking it

```bash
opencode debug agent orchestrate-local   # resolved config for one agent
opencode debug config                    # the whole merged config
models_llm                               # which model the router has resident
```
