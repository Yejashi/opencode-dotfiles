---
title: What OpenCode cannot control on the local provider
when: tuning reasoning effort or temperature per agent, a model replies with no tool call, or an agent-level option appears to do nothing
---

# Limits of the local provider

All of this was measured on the wire with a logging proxy, not inferred.

## reasoningEffort is inert for Qwen3.6

The chat template has no `reasoning_effort` hook — only `enable_thinking`. At
temperature 0, low / medium / high produced **byte-identical output** (same
sha256). The `reasoningEffort` values and the `low`/`balanced`/`deep` variants
in `opencode.json` do nothing for these models. They are harmless, but do not
reach for them to fix a quality problem.

## Agent-level sampling never leaves OpenCode

OpenCode sends only: `model`, `max_tokens`, `temperature`, `reasoning_effort`,
`stream`, `tools`, `tool_choice`. And `temperature` comes from the **model**
options (`provider.local.models.<id>.options`), not the agent. Agent-level
`options` is dropped entirely; agent-level `temperature` is ignored even with
the model-level default removed.

To change sampling, edit the model entry in `opencode.json`, or the section in
`models.ini` — not the agent.

## The failure mode to recognise: output-token exhaustion

The Qwen3.6 template primes `<|im_start|>assistant\n<think>\n`, so generation
starts **inside** the think block. With too small an output budget the model can
spend the entire allowance reasoning and never emit `</think>` or a tool call.
OpenCode sees a reply with no actions and ends the loop.

Confirmed by replay: `tokens_predicted=8192`, `stop_type=limit`, 29,894 chars,
no `</think>`.

Symptom: the agent "gives up" or returns an empty-feeling answer on a hard task,
with no error.

Fix already applied — `limit.output` is **32768** for both models in
`opencode.json` (matching the NixOS machine). `models.ini` also sets
`reasoning-budget = 8192` as a belt-and-braces cap at the llama.cpp level. If
you add a model, set both.

Cost of the larger budget: with experts partly on the CPU (the 13 GiB VRAM
cap), generation runs ~26–34 tok/s, so 32768 tokens is a ~16–21 minute
worst-case single reply. That is the trade being made.

## "All tests pass" is not verification

A benchmark run passed 36/36 while having silently rewritten a file that was
never broken, changing `pct_of(min(eligible, running), pct)` to
`min(pct_of(eligible, pct), running)` — a real bug on a path no test covered.
Diff against intent, do not just run the suite.
