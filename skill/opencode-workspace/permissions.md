---
title: Permission rules and per-agent skill rosters
when: restricting what an agent may do, giving one agent a different set of skills, or a tool asks for approval when it should not
---

# Permissions

## The evaluation rule

A permission check finds the **last matching rule** across the merged ruleset,
globbing on both the permission name and the pattern. Nothing matching means
`ask`; the base config sets `"*": "allow"` so most things are allowed.

**Last match wins, and object key order is rule order.** Broad rules first,
narrow rules last:

```json
"permission": { "bash": { "*": "ask", "git *": "allow" } }
```

Merge order is base → top-level `permission` → agent `permission`, so an agent
always overrides the global setting.

## Per-agent skill rosters

The skill listing is filtered per agent through the `skill` permission, matched
against the skill **name**. This is how one agent can see a different corpus
from another:

```json
"tester": {
  "permission": { "skill": { "*": "deny", "local-llm-ops": "allow" } }
}
```

Denying `skill` entirely also removes the whole `<available_skills>` block from
that agent's system prompt — which is why `raw` sets `"skill": false` in its
tools map. That block is injected even for agents with a custom `prompt` file,
so disabling the tool is the only way to suppress it.

## Keys that take patterns

`read, edit, glob, grep, list, bash, task, external_directory, lsp, doom_loop,
skill` accept `{ pattern: action }`. `todowrite, question, webfetch, websearch`
take a flat action only.

`external_directory` patterns are filesystem paths. Skill directories are
auto-allowed as `<dir>/*` — one level, which is why leaves stay flat.

## Checking

```bash
opencode debug agent <name>
```
