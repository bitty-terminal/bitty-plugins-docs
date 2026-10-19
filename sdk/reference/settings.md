---
title: bitty.settings Reference
description: Declared plugin settings restricted Lua configuration precedence bounds and diagnostics
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 80
---

# bitty.settings Reference

> Implementation snapshot: Core `662212c5` includes the restricted per-plugin
> configuration loader from [Core #2192](https://github.com/bitty-terminal/bitty/pull/2192).
> [Core #2198](https://github.com/bitty-terminal/bitty/pull/2198) subsequently
> merged as `1c68de2545c961790d0a2011340e4a3c04564ab7`, adding host-internal
> generation diagnostics. These source facts do not establish release or
> end-to-end plugin verification.

## Purpose and scope

`bitty.settings.get` reads the current generation's resolved settings for its
own plugin identity. The manifest `[settings]` declarations are the allowlist;
configuration supplies values, never grants. The accepted
[Lua surface RFC](../plugin-api-v1-lua-surface-rfc.md) names a settings write
direction, but Core exposes no `settings.set` bridge or `HostServices`
implementation. Plugins cannot load arbitrary user configuration files directly.

## Signature and declared keys

```lua
bitty.settings.get(key) -> value
```

Use the exact flat manifest key, such as `show_single`, without a plugin-id
prefix. Manifest keys match `^[a-z][a-z0-9_]*$` and contain at most 64 bytes.
The conceptual `plugins.<owner>.<name>` namespace does not make arbitrary dot
paths callable. Each generation receives its own map and declared-key set;
an empty declaration set permits no reads.

| Manifest kind  | Accepted value                                                        |
| -------------- | --------------------------------------------------------------------- |
| `bool`         | Lua boolean                                                           |
| `int`          | Exact Lua integer; `32.0` is not coerced to `32`                      |
| `string`       | Lua string                                                            |
| `string-array` | Dense one-based array containing only strings; `{}` is an empty array |

Manifest defaults must match their declared kind. The manifest bounds defaults
to 512 bytes per string, 32 array items, and 128 bytes per non-empty array item.
Those declaration bounds are distinct from the Lua extraction bounds below;
this page does not claim every override is revalidated against default-size
bounds. Arbitrary maps, nested settings tables, and table-shaped `entries`
have no accepted settings kind. A generic VM's ability to extract plain tables
does not extend the manifest schema.

## Configuration files and selection

The canonical user file is
`$XDG_CONFIG_HOME/bitty/plugins/<owner>.<name>.lua`; when XDG configuration is
unset, the platform configuration provider supplies the root (on Linux,
`$HOME/.config`). Package code and persistent plugin store data belong under
the data root, separately from these configuration files.

```lua
-- bitty-terminal.bar.lua: only declared settings can take effect.
return {
  show = true,
  edge = "bottom",
  show_single = true,
  name_max_chars = 24,
  modules_left = { "workspaces" },
}
```

Selection is deterministic:

1. Try the canonical `<owner>.<name>.lua` file first.
2. Only when it is absent, try `<name>.lua` if the complete current discovery
   map contains this identity, every identity is valid, and exactly one owner
   claims that short name. A missing/incomplete discovery map disables aliases;
   ambiguity never selects an arbitrary plugin. Canonical names remain usable.
3. Only when no selected Lua file exists, try the legacy
   `<owner>.<name>.toml` file as a compatibility fallback. There is no TOML
   short-name alias.

A present but rejected canonical file does not fall through to an alias or
TOML. A rejected selected alias likewise does not fall through to TOML.
Missing keys in a selected Lua file do not merge with another file.

Selection and evaluation happen during activation. The resolved values are
pinned to that generation: no configuration watcher or hot-reload path is
implemented, and a `config.reloaded` event does not re-resolve plugin settings.
Editing a file takes effect only on the next activation.

Unsafe identities never construct a configuration path. Reads reject
symlinks/reparse points, directories, FIFOs and other non-regular files,
oversize or changing files, and invalid UTF-8. Safe mode skips plugin-specific
configuration-directory reads.

## Restricted configuration VM and bounds

The host reads the selected file and evaluates it in a fresh `ConfigLuaVm`,
separate from the plugin activation VM and the main user configuration VM.
It has no host bridge, plugin services, module root, filesystem or network
access, `require`, `io`, or `os` environment/process access. It returns owned
plain data; functions, userdata, threads, metatables, cycles, invalid table
shapes, and non-finite numbers are rejected. Duplicate literal keys and
unsupported computed indexed keys are rejected before evaluation.

The loader overrides `PlainConfigLimits::source_bytes` to the plugin-file
ceiling; other limits use `PlainConfigLimits::default` at the inspected Core
revision:

| Limit / source constant                           | Current value                        |
| ------------------------------------------------- | ------------------------------------ |
| Source bytes / `MAX_PLUGIN_FILE_BYTES`            | 16 KiB                               |
| Instructions / `RC1_INSTRUCTION_BUDGET`           | 10,000,000                           |
| Charged wall time / `RC1_WALL_CLOCK_BUDGET_MS`    | 50 ms                                |
| Accounted VM heap / `RC2_MEMORY_PER_PLUGIN_BYTES` | 32 MiB                               |
| Extracted nesting / `max_depth`                   | 16                                   |
| Entries per extracted table / `max_keys`          | 64                                   |
| Extracted values / `max_values`                   | 256                                  |
| Bytes per extracted string / `max_string_bytes`   | 2048                                 |
| Aggregate extracted bytes / `max_aggregate_bytes` | 16 KiB including structural overhead |

These are bounded implementation values, not authority granted by a config
file. The legacy TOML parser remains a separate flat data-only subset with
its own line, key, string, and array bounds in `bitty-config::plugin_file`.

## Precedence and failure recovery

For each declared key, resolution proceeds from lowest to highest:

```text
manifest default < shared host settings source < selected user file
```

The terminal integration supplies `EmptySettings` as the shared source in this
snapshot; do not infer a populated main-config plugin table from the resolver
seam. Wrong-typed or unsupported values drop only that entry, leaving the next
valid lower layer. Undeclared keys are dropped and cannot create settings or
capabilities. Structural, syntax, resource, or whole-file extraction failures
reject the selected file and use lower layers for every declared key. An
absent file is ordinary default behavior, not an error. Rejection itself does
not abort activation; a plugin that makes an unguarded undeclared read can
still fail activation.

## Results and errors

A normally resolved declared key has a shipped default and returns a typed
value. At the `HostServices::settings_get` seam, `Ok(None)` maps to Lua `nil`
when a permitted key has no backing value. This does not make an undeclared
read return `nil`.

Denials are catchable error tables with `class`, `code`, and `message`:

| Code                     | Class        | Condition                                                                 |
| ------------------------ | ------------ | ------------------------------------------------------------------------- |
| `E_SETTINGS_KEY_INVALID` | `validation` | Non-string bridge argument, or runtime key empty or longer than 128 bytes |
| `E_SETTINGS_UNDECLARED`  | `validation` | Key absent from this generation's manifest declarations                   |
| `E_TIMEOUT`              | `budget`     | Bounded bridge read exceeded its deadline                                 |

No capability is required for an allowed own-namespace read. Settings access
does not imply filesystem, process, network, notification, or cross-plugin
authority.

## Generation diagnostics

At merged Core `1c68de25`, the runtime retains at most
`MAX_PLUGIN_SETTINGS_DIAGNOSTICS = 4` host-internal diagnostics per activation:
ambiguous alias with claimant count, canonical-over-alias selection, count of
ignored entries, and selected-file rejection with lower-layer fallback.
The terminal startup activation path emits these through its warning logger.
Warnings omit setting names, values, file contents, and raw evaluator errors;
validated short aliases and counts may appear. Runtime diagnostics are owned
by the generation and reset before a new activation. This adds no Lua diagnostics API
and changes neither public `ActivationReport` nor `LoadedPluginSettings`
layouts.

## Official plugin reconciliation

[Registry #92](https://github.com/bitty-terminal/bitty-plugins/pull/92), merged
as `ba4d9d1268bfd877ab60413f496e95b6cae41168`, pins activity, bar,
file-manager, git-panel, and palette at `>=0.0.24,<0.1` / Plugin API `^1.0`.
Eligibility and manifest hashing do not complete
[official schema issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90),
which remains open at this snapshot:

- [Activity](../../docs/plugins/activity/schemas.md): `retention_days` and
  `store_command_args` reads are undeclared. Guarded reads use local defaults;
  `store_command_args` does not enable argument storage even if true.
- [Bar](../../docs/plugins/bar/schemas.md): chassis scalars and module-group
  string arrays are declared. Per-module option keys are declared as empty
  string arrays; map-shaped options remain inert under the strict schema.
- [File manager](../../docs/plugins/file-manager/schemas.md) and
  [palette](../../docs/plugins/palette/schemas.md): settings reads are
  undeclared and guarded. Table-shaped `entries` remains explicitly deferred;
  this reference accepts no replacement schema.
- Git-panel has no `settings.get` reads. Composer's five chord/role strings,
  copy-mode/search `row_count` and `max_bytes`, and statusline's six formatting
  keys still need declarations in the inspected sources. Their compatibility
  and activation status must not be inferred from the five eligible pins.
- Devtools has no settings-read gap in the inspected source, but was not one
  of those five eligible pins. Its other integration work remains pending.

Wheel's project-config loader and AI behavior remain separate follow-ups in
issue #90; they do not broaden this configuration VM or its allowed settings kinds.

## Compatibility and versioning

This draft describes inspected implementation and keeps outstanding schema
work explicit. Configuration values never change manifest-hash-bound grants.
The accepted API's compatibility target remains versioned separately from
the application version; neither the API range nor registry inclusion proves
that a plugin is usable or shipped.

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
- [Plugin Host Runtime RFC](../../runtime/plugin-host-runtime-rfc.md)
- [Plugin package management](../../extensibility/package-management.md)
- [Lua reference overview](overview.md)
- Core `662212c5`: `crates/bitty-plugin-host/src/manifest.rs` (`SettingDecl`,
  `SettingKind`); `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (`settings_get`); `plugin_settings.rs` (selection and merge);
  `crates/bitty-lua/src/plain_config.rs` and `stdlib.rs` (restricted VM).
- Core `1c68de25`: runtime generation diagnostic retention,
  `crates/bitty-terminal/src/plugin_runtime.rs` warning integration, and
  `crates/bitty-runtime/tests/plugin_settings_file.rs` regression tests.
- [Official schema follow-up #90](https://github.com/bitty-terminal/bitty-plugins/issues/90)
- [Documentation follow-up #153](https://github.com/bitty-terminal/bitty-plugins-docs/issues/153)
- Additional inspected sources: statusline `2b9973c7`, devtools `c28b9dab`,
  composer `eb05a8fc`, copy-mode `edeae30e`, and search `a3631882`. The latter
  three are registry `ba4d9d1` pins, not updated eligibility claims.
