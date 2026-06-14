# CLAUDE.md

This repo holds instance definitions consumed by the Plasmo Launcher's **metaserver**. Metaserver clones this repo via the GitHub API on startup (and on webhook push), walks each top-level directory, resolves mod versions against Modrinth, and serves the assembled `MetaInstance` records to the launcher client.

The launcher client never reads this repo directly — it only sees the metaserver's resolved JSON output.

## Layout

Each **top-level directory** is one instance. Directory name = instance key. Dot-prefixed dirs are skipped.

```
<instance>/
  Instance.toml          # required
  mods/                  # optional
    <mod>.toml
    optional/
      <mod>.toml
  files/                 # optional
    Meta.toml            # optional, per directory
    <arbitrary tree>
```

## `Instance.toml`

```toml
name="Test"             # display name (any string)
game_version="26.1.2"   # exact Minecraft version string
[loader]
type="Fabric"           # "Fabric" or "Vanilla"
version="0.19.3"        # required for Fabric; omit for Vanilla
```

`game_version` and `loader.type` are passed verbatim to Modrinth as the filter when resolving each mod, so they must match what Modrinth labels the version as (e.g. `"26.1.2"`, not `"26.1"`).

## `mods/<mod>.toml`

```toml
id="sodium"             # Modrinth project slug or ID
provider="modrinth"     # currently the only supported provider
version="release"       # optional, default "latest"
recommended=false       # optional, default false (only meaningful for optional mods)
category="None"         # optional: "None" | "Test"

[locale.en]             # optional per-language display info
title="Sodium"
description="Performance mod"
```

`version` accepts either an **implicit channel** (`"latest"`, `"release"`, `"beta"`, `"alpha"`) or an **explicit version string** (e.g. `"0.6.13"`). Implicit channels resolve to the newest Modrinth version on that channel matching the instance's `game_version` + loader.

Mods placed directly under `mods/` are **required**. Mods placed under `mods/optional/` are surfaced as optional — the launcher lets the user toggle them.

Any `.toml` that fails to parse, refers to a missing project, or has no matching version for the instance's game version/loader is **silently skipped with a warning in metaserver logs** — the instance still loads without that mod. Check `cargo run -p plasmolauncher-metaserver` output when debugging missing mods.

## `files/`

Arbitrary file tree shipped alongside the instance (configs, default options, resource packs, etc.). Mirrored into the instance directory on the client.

Each subdirectory may contain a `Meta.toml`:

```toml
validate = ["options.txt", "servers.dat"]
```

Files listed in `validate` are re-checked against the metaserver hash on every launch and overwritten if the user's local copy diverges. Files **not** listed are written only once (first install) and treated as user-owned afterward.

## Adding a new instance

1. `mkdir <name>/` at the repo root.
2. Write `<name>/Instance.toml` with the MC version + loader.
3. Add `<name>/mods/*.toml` for any required mods (and `mods/optional/*.toml` for optional ones).
4. Optionally drop config defaults under `<name>/files/`.
5. Commit + push. Metaserver picks up the change on webhook (or restart).

When in doubt about the schema, the authoritative types live in the monorepo:

- `common/src/models/instance.rs` — `Instance`, `Loader`
- `metaserver/src/models/mod_toml.rs` — `ModToml`, `ModVersion`
- `metaserver/src/models/files_meta_toml.rs` — `FilesMeta`
- `metaserver/src/generate_meta/` — the walker that turns this repo into `MetaInstance` JSON
