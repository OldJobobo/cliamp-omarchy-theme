# AGENTS.md — cliamp-omarchy-theme

## Project Overview

This repository provides a **cliamp** (terminal audio player) theme integration for the **Omarchy** desktop environment. It consists of:
- A TOML theme file (`cliamp.toml`) defining 6 color keys
- A mustache template (`cliamp.toml.tpl`) for dynamic theme rendering
- A bash hook (`50-cliamp.sh`) that deploys the theme on Omarchy theme changes

cliamp themes: 6-key TOML files in `~/.config/cliamp/themes/<name>.toml`.
Keys: `accent`, `bright_fg`, `fg`, `green`, `yellow`, `red`.

## Build / Test / Validation Commands

This repo has **no build step, no tests, no linter**. It is a passive theme asset.

### Validation
```bash
# Verify TOML is valid
python3 -c "import tomllib; tomllib.load(open('cliamp.toml', 'rb'))"

# Verify the template has valid mustache syntax (no unclosed {{ }})
grep -c '{{' cliamp.toml.tpl && grep -c '}}' cliamp.toml.tpl

# Verify bash script syntax
bash -n 50-cliamp.sh
```

### Deployment (live system)
```bash
chmod +x ~/.config/omarchy/hooks/theme-set.d/50-cliamp.sh
omarchy theme set <theme-name>
```

---

## Code Style Guidelines

### Shell Scripts (`.sh`)

- Use `#!/bin/bash` (not `#!/bin/sh`)
- Use `command -v foo >/dev/null 2>&1` for existence checks
- Use `skipped` / `success` / `error` functions from Omarchy hook framework (do not use bare `echo`)
- Always `exit 0` on success; hooks may rely on exit codes
- Guard `sed -i` with `grep -q` checks to avoid redundant writes
- Always `mkdir -p` before writing into a directory
- Use `"$HOME/.config/..."` paths; never hardcode `/home/username`
- Prefer `cp -f` over `cat > file << EOF` for copying generated files

### TOML Theme Files

- Keys must appear in this **canonical order**: `accent`, `bright_fg`, `fg`, `green`, `yellow`, `red`
- Hex colors: uppercase or lowercase `#RRGGBB` (be consistent within a file)
- No trailing commas, no comments in the TOML values themselves
- One space around `=`: `key = "#RRGGBB"`

### Mustache Templates (`.tpl`)

- Use `{{ variable }}` syntax (mustache, double braces)
- Template variables map to Omarchy `colors.toml` keys:
  - `{{ accent }}`, `{{ color15 }}`, `{{ color7 }}`, `{{ color2 }}`, `{{ color3 }}`, `{{ color1 }}`
- Wrap in TOML comment header explaining the color mapping

### Bash Hook Variables (from Omarchy theme-set)

Omarchy exports these color variables to hook scripts:
```
${normal_black}   ${normal_red}     ${normal_green}   ${normal_yellow}
${normal_blue}    ${normal_magenta} ${normal_cyan}    ${normal_white}
${bright_black}   ${bright_red}     ${bright_green}  ${bright_yellow}
${bright_blue}    ${bright_magenta} ${bright_cyan}    ${bright_white}
${accent}
```
Note: `accent` is **not** exported by `theme-set` hooks; use `${normal_blue}` as the closest equivalent for cliamp's `accent` key. The template path uses `{{ accent }}` directly (available in `colors.toml`).

### Naming Conventions

- Theme TOML files: lowercase with hyphens (e.g., `watchmen.toml`)
- Hook scripts: `NN-name.sh` where `NN` is load order (40-cava.sh, 50-cliamp.sh, etc.)
- Target theme name in cliamp: `omarchy.toml`

### File Placement

- **Template files** (`*.tpl`): place in repo root so Omarchy's TPL renderer picks them up
- **Hook scripts** (`50-cliamp.sh`): live at `~/.config/omarchy/hooks/theme-set.d/`
- **Generated theme**: `~/.config/cliamp/themes/omarchy.toml`

### Error Handling

- Hook scripts: **never** `exit 1` for missing dependencies — use `skipped "dependency-name"` and `exit 0`
- Do not overwrite config files aggressively; always use grep guards before `sed -i`
- Validate TOML structure before writing if generating from scratch

---

## Color Mapping Reference

| cliamp key   | Omarchy colors.toml | Bash var            | Purpose                              |
|--------------|---------------------|---------------------|--------------------------------------|
| `accent`     | `accent` / `color4` | `${normal_blue}`    | Interactive highlights               |
| `bright_fg`  | `color15`           | `${bright_white}`   | Primary text / time display           |
| `fg`         | `color7`            | `${normal_white}`   | Muted text / help bar                |
| `green`      | `color2`            | `${normal_green}`   | Playing indicator / volume bar        |
| `yellow`     | `color3`            | `${normal_yellow}`  | Mid-spectrum gradient                |
| `red`        | `color1`            | `${normal_red}`     | High spectrum / errors               |

---

## Related Repositories

- **Omarchy core**: `~/.config/omarchy/`
- **cliamp config**: `~/.config/cliamp/`
- **Omarchy themes vault**: `~/Projects/themes/`
- **Omarchy template substitution**: `~/Projects/oldjobobo-custom-omarchy-templates/`
- **Base16 schemes**: `~/Projects/schemes/base16/`
