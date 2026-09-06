---
title: Capture the exact prompt the model received
when: the model behaves oddly, ignores instructions, refuses, or you need to measure how large a system prompt actually is
---

# Prompt logging

`start-router.sh` supports `--log-prompts-dir` behind an environment variable.
It is off unless set, because it writes every rendered prompt — system prompt,
tool schemas, full history — to disk in the clear.

```bash
stop_llm
LLM_ROUTER_LOG_PROMPTS=/tmp/llm-prompts /home/yejashi/local-ai/router/start-router.sh
```

Then, from another terminal, run the request you want to inspect and read the
dump:

```bash
ls -t /tmp/llm-prompts | head
wc -c /tmp/llm-prompts/*        # how big the system prompt really is
```

This is the tool that settled the "heretic model seems censored" question: it
showed OpenCode's `build` agent injecting ~32.5 KB of system prompt versus
~2 KB for the `raw` agent. When a local model misbehaves, read what it was
actually sent before concluding anything about the weights.

Turn it off when done — the dumps contain whole conversations:

```bash
rm -rf /tmp/llm-prompts
start_llm
```

Lighter-weight alternatives that do not dump content:

```bash
logs_llm                  # load/unload, slot, and timing lines
models_llm                # which model is resident
```
