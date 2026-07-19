# test

A performance-and-QoL Fabric baseline derived from [Fabulously Optimized](https://download.fo). Useful as a starting point — copy this directory, rename it, and edit the files below.

For the schema (`Instance.toml`, `mods/*.toml`, `Meta.toml`), see the repo-root [CLAUDE.md](../CLAUDE.md).

## Files specific to this instance worth reviewing when forking

### Branding (swap for your modpack identity)
- **`files/config/crash_assistant/config.toml`** — `help_link`, `support_name`, `support_place`, `modpack_name`. Shown to users when the game crashes.
- **`files/config/isxander-main-menu-credits.json`** — text + URL in the main-menu bottom-right corner.

### Opinionated defaults you may want to retune
- **`files/options.txt`** — vanilla MC settings (render distance, GUI scale, gamma, keybinds). Most likely to clash with personal preference.
- **`files/debug-profile.json`** — which lines appear in the F3 overlay.
- **`files/config/*`** — per-mod configs. Each file overrides specific mod defaults; check git history for what each changes.
