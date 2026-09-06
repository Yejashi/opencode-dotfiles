# opencode-dotfiles

My [OpenCode](https://github.com/opencode-ai/opencode) configuration — the agents, prompts, and settings I use to shape how the AI assistant behaves.

## What's here

- **opencode.json** — Main config. Defines the local LLM provider, LSP (ruff), and four agent definitions (`raw`, `orchestrate`, `implementer`, `tester`).
- **opencode.jsonc** — Minimal placeholder config.
- **prompts/** — Prompt files that each agent loads via `{file:...}` references.

| Agent | Role |
|---|---|
| `raw` | Unfiltered chat / creative writing. No tools, no coding-agent system prompt. |
| `orchestrate` | Plans and delegates to subagents. Never edits code directly. |
| `implementer` | Executes one specific, already-decided code change. |
| `tester` | Runs commands and reports pass/fail. Never edits files. |

## Setup

1. Clone this repo into your OpenCode config directory:
   ```bash
   mkdir -p ~/.config/opencode
   cp -r opencode-dotfiles/* ~/.config/opencode/
   ```
2. Install the plugin dependency:
   ```bash
   cd ~/.config/opencode && npm install
   ```
3. Edit `opencode.json` and replace `"YOUR_API_KEY_HERE"` with your actual API key for your local LLM endpoint.
4. Restart OpenCode.

## Notes

- This repo is intentionally minimal — it contains only config and prompt files. Dependencies (`node_modules/`) are excluded.
- The `AGENTS.md` instruction file lives in your home directory, not in this repo.

## License

Feel free to fork and adapt.
