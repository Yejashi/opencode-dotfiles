# opencode-dotfiles

My [OpenCode](https://opencode.ai) configuration for the Arch machine. It is
the same setup my NixOS machine deploys through home-manager
(`nixOS-Configuration/users/yejashi/opencode`), adapted only where the hardware
differs.

This repo **is** `~/.config/opencode` — the directory is a symlink to it, so
every change made here (or by OpenCode itself) lands in the working tree and
just needs a commit and push.

## Layout

| Path | What it is |
| --- | --- |
| `opencode.json` | Providers, models, LSP, agents, server plugin |
| `tui.json` | TUI plugin (token tracker), keybinds, attention |
| `prompts/` | Agent prompts, loaded via `{file:./prompts/<name>.md}` |
| `plugins/token-tracker.tsx` | Sidebar footer with live token/speed/cost stats |
| `themes/ayu-dark.json` | TUI theme |
| `ORCHESTRATION.md` | How the orchestrator/worker split works |
| `skill/` | Hub-style skills (see `skill/README.md`) |
| `command/` | harness-memory slash commands |
| `bin/` | `skills-build`, `skills-import-awesome`, `harness-memory-repatch` |

## Agents

| Agent | Mode | Model | Tools |
| --- | --- | --- | --- |
| `orchestrate-local` | primary | local Qwen | glob, grep, webfetch, task, todowrite |
| `orchestrate-frontier` | primary | `openai/gpt-5.6-terra` | same as above |
| `raw` | primary | local Qwen heretic | none |
| `explore` | subagent | local Qwen | read, glob, grep |
| `implementer` | subagent | local Qwen | read, write, edit, patch |
| `operator` | subagent | local Qwen | bash (state-changing, exact commands) |
| `tester` | subagent | local Qwen | bash (read-only inspection, builds, tests) |

Both orchestrators may only delegate to the four workers. Title, summary, and
compaction also run locally, so the frontier API never sees worker tool loops.
See `ORCHESTRATION.md`.

## Setup on Arch

```bash
git clone git@github.com:Yejashi/opencode-dotfiles.git ~/Documents/repos/opencode-dotfiles
ln -s ~/Documents/repos/opencode-dotfiles ~/.config/opencode

cd ~/.config/opencode
npm install                  # harness-memory + @opencode-ai/plugin
bin/harness-memory-repatch   # re-apply local fixes (quiet TUI logging etc.)
```

Needed on `PATH` for the LSP block: `ruff`, `clangd`, `bash-language-server`.

For `orchestrate-frontier`, export the key from private shell config (never
from this repo):

```bash
export OPENAI_API_KEY="..."
```

## Differences from the NixOS copy

Shared verbatim: `prompts/`, `plugins/`, `themes/`, `ORCHESTRATION.md`,
`opencode.jsonc`, and the whole agent roster. `opencode.json` matches too —
same 128k context, output, and compaction settings — except:

- **VRAM cap (server side, not in this repo)** — the same IQ4_XS models, but
  `~/local-ai/router/models.ini` keeps each one under 13 GiB of VRAM with
  `n-cpu-moe` (the expert weights of 16 layers for base, 17 for heretic, live
  in system RAM), so generation runs ~33–35 tok/s. The base model also uses
  n-gram speculative decoding, which makes rewriting existing text ~4x faster. Its `ctx-size = 131072`
  must match `limit.context`. The router unit's `MemoryHigh` is 16G to hold
  those experts without throttling.
- **Router API key** — `~/local-ai/router/start-router.sh` reads
  `.provider.local.options.apiKey` from this file with `jq` and passes it to
  `llama-server --api-key`. Keep that key path intact.
- **`raw` uses the heretic model**, which only the Arch router serves.
- **harness-memory** loads from `./node_modules` (installed by `npm install`,
  patched by `bin/harness-memory-repatch`) instead of the NixOS package-cache
  path, because this directory is writable.
- **Extra skills and commands** under `skill/` and `command/`.

To check the shared files for drift:

```bash
nix=~/Documents/repos/nixOS-Configuration/users/yejashi/opencode
for p in prompts plugins themes ORCHESTRATION.md opencode.jsonc; do
  diff -r "$nix/$p" ~/.config/opencode/"$p"
done
```

## Not in git

`node_modules/`, lockfiles, backups, and the machine-local story downloader
(`bin/story-dl`, `story-dl.ini`, `skill/story-archive/`).
