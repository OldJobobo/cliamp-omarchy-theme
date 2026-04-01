# cliamp-omarchy-theme

Automatic [cliamp](https://github.com/bjarneo/cliamp) theme integration for the [Omarchy](https://github.com/basecamp/omarchy) desktop environment.

## What it does

Every time you switch Omarchy themes, this hook regenerates a matching `omarchy.toml` theme for cliamp, keeping your audio visualizer colors in sync with your desktop.

## Components

| File | Purpose |
|------|---------|
| `50-cliamp.sh` | Omarchy theme-set hook — deploys the theme |
| `cliamp.toml.tpl` | Mustache template for live Omarchy theme rendering |
| `cliamp.toml` | Reference theme (Watchmen palette) |

## Color Mapping

| cliamp key | Purpose |
|------------|---------|
| `accent` | Interactive highlights |
| `bright_fg` | Primary text / time display |
| `fg` | Muted text / help bar |
| `green` | Playing indicator / volume bar |
| `yellow` | Mid-spectrum |
| `red` | High spectrum / errors |

## Installation

```bash
# Install the hook
chmod +x 50-cliamp.sh
cp 50-cliamp.sh ~/.config/omarchy/hooks/theme-set.d/

# Deploy the current Omarchy theme as cliamp's default
omarchy theme set "$(basename "$(readlink ~/.config/omarchy/current/theme)")"
```

cliamp must already have `~/.config/cliamp/config.toml` configured (with or without an existing `theme` value).

## How it works

The hook runs on every `omarchy-theme-set`. It reads the Omarchy theme's `colors.toml` (or the generated template), maps those colors to the 6 cliamp keys, and writes the result to `~/.config/cliamp/themes/omarchy.toml`. It also ensures `theme = "omarchy"` is set in cliamp's config.

Since cliamp reads its theme at startup, you may need to restart cliamp after the first run or after a theme switch.

## Requirements

- [Omarchy](https://github.com/basecamp/omarchy)
- [cliamp](https://github.com/bjarneo/cliamp/)
