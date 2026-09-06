---
title: LSP servers and the lsp config key
when: language diagnostics are missing, adding a language server, or editing the lsp key in opencode.json
---

# LSP

## The rule that blocks people

Omitting `lsp` entirely, or setting it to `false`, **disables LSP completely**.
It is not on by default — installing language servers alone does nothing. `true`
enables the built-ins; **an object also enables all built-ins**, plus whatever it
overrides.

## Installed here (from Arch repos, not auto-downloaded)

| Package | Binary | Purpose |
| --- | --- | --- |
| `bash-language-server` | bash-language-server | Bash completion/hover/definition |
| `clang` | clangd 22.1.8 | C/C++ |
| `pyright` | pyright-langserver | Python types + diagnostics |
| `ruff` | ruff | Python lint diagnostics/format |
| `shellcheck` | shellcheck | Bash **diagnostics** backend |

`bash-language-server` produces no error diagnostics without `shellcheck` — it
only supplies completion and hover.

## The config, and why it is this small

Built-in ids include `bash`, `clangd`, `pyright`. There is **no built-in
`ruff`**, so it is the only entry defined:

```json
"lsp": { "ruff": { "command": ["ruff", "server"], "extensions": [".py", ".pyi"] } }
```

**Do not replace the built-in bash/clangd/pyright entries with explicit
`command` arrays.** Overriding `command` discards the built-in spawn logic:

- built-in `pyright` detects `VIRTUAL_ENV` / `.venv` / `venv` and passes
  `pythonPath` via `initialization`;
- built-in `clangd` resolves the project root from `compile_commands.json`,
  `compile_flags.txt`, or `.clangd`, and passes `--background-index --clang-tidy`.

An earlier revision hardcoded those and silently lost both behaviours.

## clangd without a compile database

`~/.config/clangd/config.yaml` is a global fallback (`-std=c++20 -Wall -Wextra`,
`Diagnostics.UnusedIncludes: Strict`) that applies only when a project has no
`compile_commands.json` / `compile_flags.txt` / `.clangd`. For real C++ work,
generate a compile database — clangd's include resolution is poor without one:

```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ...   # or `bear -- make`, but bear is not installed
```

```bash
opencode debug lsp
```
