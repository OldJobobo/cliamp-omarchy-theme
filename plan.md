# Plan: cliamp Omarchy Theme Integration

## Context

cliamp (`~/.local/bin/cliamp`, v1.31.5) is installed and configured with Jellyfin but using the built-in default theme (`theme = ""` in `config.toml`). This plan adds a hooklett and TPL so every Omarchy theme change regenerates a matching cliamp theme automatically.

cliamp themes: 6-key TOML files in `~/.config/cliamp/themes/<name>.toml`. Keys: `accent`, `bright_fg`, `fg`, `green`, `yellow`, `red`.

---

## Color Mapping

| cliamp key  | Omarchy colors.toml key | Omarchy bash var    | Rationale |
|-------------|------------------------|---------------------|-----------|
| `accent`    | `accent` / `color4`    | `${normal_blue}`    | TPL uses `{{ accent }}`; hooklett uses `${normal_blue}` (closest exported var — `accent` is not exported by theme-set) |
| `bright_fg` | `color15`              | `${bright_white}`   | Brightest foreground, max legibility for primary text/time display |
| `fg`        | `color7`               | `${normal_white}`   | Standard foreground, one step dimmer — muted text, help bar |
| `green`     | `color2`               | `${normal_green}`   | Semantic match — playing indicator, volume bar |
| `yellow`    | `color3`               | `${normal_yellow}`  | Semantic match — mid-spectrum gradient |
| `red`       | `color1`               | `${normal_red}`     | Semantic match — top spectrum, errors |

---

## Files to Create

### 1. TPL Reference File

**Path:** `~/Projects/oldjobobo-custom-omarchy-templates/reference/cliamp.toml.tpl`

Place in `reference/` subdirectory (NOT repo root) to prevent `scripts/install.sh` from symlinking it to `~/.config/omarchy/themed/` — the Omarchy TPL renderer would map it to `~/.config/cliamp/cliamp.toml` (wrong path; correct target is `~/.config/cliamp/themes/omarchy.toml`).

Uses `{{ }}` mustache syntax matching the `colors.toml` variable names. Uses `{{ accent }}` directly (available in colors.toml and confirmed in `colors.css.tpl`).

### 2. Hooklett

**Path:** `~/.config/omarchy/hooks/theme-set.d/50-cliamp.sh`

Follows the exact `40-cava.sh` pattern. Numbered `50-` to match `50-heroic.sh` tier.

Key design decisions:
- No staging file (unlike cava) — cliamp has no live-reload signal, overwrite directly every run
- No `require_restart` — cliamp reads theme at startup; notify-send would be misleading
- `sed` targets `^theme = ".*"` to match `theme = ""` and any future theme name
- grep guard prevents redundant sed calls
- `accent` uses `${normal_blue}` since `accent` is not exported by theme-set

---

## Deployment

```bash
chmod +x ~/.config/omarchy/hooks/theme-set.d/50-cliamp.sh
omarchy theme set $(basename $(readlink ~/.config/omarchy/current/theme))
```

**Verify:**
- `~/.config/cliamp/themes/omarchy.toml` exists with 6 hex color entries
- `~/.config/cliamp/config.toml` has `theme = "omarchy"`
- Terminal shows `[SUCCESS] cliamp theme updated!`
