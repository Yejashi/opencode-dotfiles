---
name: opencode-workspace
description: Use when working on this machine's own OpenCode setup — the raw/orchestrate/implementer/tester agents, which model an agent runs on, authoring or debugging skills, permission rules, LSP config, the MEMORY.md protocol, or why a config change did not take effect. Use ONLY for configuring OpenCode itself, not for the user's application code.
---

# This machine's OpenCode workspace

Config lives in `~/.config/opencode/`: `opencode.json`, agent prompts under
`prompts/`, skills under `skill/`, helper scripts under `bin/`.

Two rules that cause most confusion here:

1. **Config is not hot-reloaded.** Every change needs a full restart.
2. **The local model is the constraint.** Anything below that assumes a ~35B
   IQ3 quant at 64k context, not a frontier model.

For generic OpenCode schema questions the built-in `customize-opencode` skill is
authoritative. This skill covers what is specific to *this* setup and what was
learned by measuring it.

<!-- BEGIN GENERATED INDEX -->

## Topics in this skill

Each entry below is a file in `/home/yejashi/.config/opencode/skill/opencode-workspace`. Read the one whose *use when* matches the task — do not read them all. This list is complete and authoritative; ignore the sampled file list appended by the skill tool.

- **The agent roster and when to use each** — use when: choosing between build, orchestrate, raw, implementer, or tester; editing an agent definition; or deciding whether delegation is worth it
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/agent-roster.md`
- **Add or edit a skill in this workspace** — use when: writing a new skill, adding a topic to an existing one, or the hub index looks out of date
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/authoring-skills.md`
- **Change opencode.json safely** — use when: editing OpenCode config, a change appears to have no effect, or OpenCode refuses to start after an edit
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/config-changes.md`
- **What OpenCode cannot control on the local provider** — use when: tuning reasoning effort or temperature per agent, a model replies with no tool call, or an agent-level option appears to do nothing
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/local-model-limits.md`
- **LSP servers and the lsp config key** — use when: language diagnostics are missing, adding a language server, or editing the lsp key in opencode.json
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/lsp-config.md`
- **The MEMORY.md protocol** — use when: finishing significant work, recording a decision or blocker, or looking for prior context on this machine
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/memory-protocol.md`
- **Which model an agent actually runs on** — use when: an agent runs on an unexpected model, you are adding or removing an agent's model pin, or you want subagents to follow the picker
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/model-resolution.md`
- **Permission rules and per-agent skill rosters** — use when: restricting what an agent may do, giving one agent a different set of skills, or a tool asks for approval when it should not
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/permissions.md`
- **How OpenCode finds and loads skills** — use when: a skill is not offered to the model, you are deciding where to put one, or you are worried about skills bloating the system prompt
  read `/home/yejashi/.config/opencode/skill/opencode-workspace/skill-system.md`

<!-- END GENERATED INDEX -->
