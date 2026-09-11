---
title: A local model refuses or deflects
when: an uncensored or abliterated model still refuses, gives hedged non-answers, or behaves differently than it does outside OpenCode
---

# Refusals are usually the system prompt, not the weights

This was diagnosed here directly, by A/B testing identical weights and sampler
settings: the same request **complied with no system prompt and refused with
OpenCode's**.

Abliteration strips the learned refusal direction from the weights. It does
nothing about a role imposed in context. OpenCode's `build` agent injects
~32.5 KB — mostly tool schemas, plus "You are opencode, an interactive CLI tool
that helps users with software engineering tasks", an instruction to answer in
fewer than four lines, and:

> "If you cannot or will not help the user with something, do not say why …
> offer helpful alternatives"

which literally scaffolds a polite deflection. The model is playing the role it
was handed.

## The fix already in place

The `raw` agent in `~/.config/opencode/opencode.json`:

- `prompt` → `prompts/raw.md`, a minimal non-coding persona
- every tool disabled
- pinned to `local/qwen3.6-35b-a3b-heretic`

System prompt drops from ~32.5 KB to ~2 KB and the model complies. Reach it
with `opencode --agent raw`, or Tab in the TUI. The `build` agent is untouched
for coding work.

## Diagnosing a new case

1. Confirm the model actually loaded (`models_llm`) — a swap failure can look
   like a personality change.
2. Dump the real prompt (see the prompt-logging topic) and read it.
3. Compare against a bare request with no system prompt at all.
4. Only after those three should you suspect the quant or the weights.

Note that switching to `raw` forces a model swap, because it is pinned to the
heretic quant and only one model is resident at a time. Expect the ~15 s load.
