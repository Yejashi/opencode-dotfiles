---
title: Install and inspect packages
when: a command is not found, installing a tool, or checking what version of something is on this machine
---

# Packages

Arch, `pacman` for the official repos and `yay` for the AUR (`paru` is not
installed).

```bash
pacman -Q <pkg>            # installed? which version?
pacman -Qo $(command -v X)  # which package owns this binary
pacman -F <file>           # which package would provide it (needs pacman -Fy)
pacman -Ss <term>          # search official repos
```

Installing needs root, so it is a **user action, not an agent action** — print
the command and let the user run it:

```bash
sudo pacman -S <pkg>       # official
yay -S <pkg>               # AUR
```

## What this stack depends on

| Package | Why |
| --- | --- |
| `bash-language-server`, `shellcheck` | Bash LSP; shellcheck supplies the diagnostics |
| `clang` | clangd 22.1.8, the C/C++ LSP |
| `pyright`, `ruff` | Python LSP and linter |
| `hip-runtime-amd` | HIP runtime for the llama.cpp ROCm build |
| `jq` | `start-router.sh` and `models_llm` both parse config with it |

## Notably *not* installed

`rocm-smi`, `rocminfo`, `amd-smi`, `radeontop`, `amdgpu_top`, `nvtop`, `bear`,
`paru`. Commands referencing these will fail — use the sysfs alternatives in the
GPU topic, or ask the user to install first.

`~/.bashrc` also sources a Spack environment
(`~/Documents/repos/spack/share/spack/setup-env.sh`), so some tools on PATH in
an interactive shell come from Spack rather than pacman. A non-interactive shell
will not have them.
