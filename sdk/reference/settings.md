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
> generation diagnostics. This page was re-inspected read-only at `bitty`
> origin/main `b5145675`; the values and paths below are unchanged there. These
> source facts do not establish release or end-to-end plugin verification.

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
prefix. Manifest keys match `^[a-z][a-z0-9_]*$` and contain at most 64 bytes,
and one manifest declares at most 32 keys (`MAX_SETTINGS_KEYS`). The
conceptual `plugins.<owner>.<name>` namespace does not make arbitrary dot
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

## Legacy TOML compatibility fallback

`$XDG_CONFIG_HOME/bitty/plugins/<owner>.<name>.toml` remains a data-only
compatibility layer; there is no short-name TOML alias, and the file is
consulted only when neither Lua file exists. It is parsed by
`bitty-config::plugin_file` and is never evaluated as code: no includes, no
shell, no nesting, and no table sections.

The accepted subset is flat keys and four value forms:

```text
show = true                 # boolean
name_max_chars = 32         # bare integer, optional leading `-`
edge = "bottom"             # double-quoted string with \\ \n \t \r escapes
modules = ["a", "b"]        # single-line array of double-quoted strings
```

- Keys are bare `[A-Za-z0-9_.-]+` or double-quoted. A `[section]`, a line
  without `=`, or a malformed key rejects the whole file; duplicate keys
  reject the whole file as well, because silent shadowing would be a side
  channel.
- Floats, quoted numbers, inline tables, nested arrays, and multi-line values
  are outside the subset: the offending key falls back to its lower layer
  while the rest of the file still applies. An unterminated string or array
  skips that key only.
- Unknown keys are parsed and then dropped at merge time, recorded as
  skipped. A `[capabilities]`-looking key grants nothing.

Bounds are enforced before any value is trusted (`plugin_file.rs:140-158` at
the inspected revision):

| Limit / source constant                       | Value                    |
| --------------------------------------------- | ------------------------ |
| `MAX_PLUGIN_FILE_BYTES`                       | 16 KiB                   |
| `MAX_PLUGIN_FILE_LINES`                       | 512                      |
| `MAX_PLUGIN_LINE_BYTES`                       | 1024 bytes per line      |
| `MAX_PLUGIN_FILE_KEYS`                        | 64 keys per file         |
| `MAX_PLUGIN_STRING_BYTES`                     | 512 bytes per string     |
| `MAX_PLUGIN_ARRAY_ITEMS`                      | 32 items per array       |
| `MAX_PLUGIN_ARRAY_ITEM_BYTES`                 | 128 bytes per array item |
| Non-UTF-8 file, NUL in a string, or bad value | Rejected, no lossy read  |

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

The official registry at `bitty-plugins` `2da1c87` carries twelve entries under
`registry/official/`; every `manifest_hash` equals the sha256 of that plugin's
pinned submodule `bitty-plugin.toml`, checked at the pin. All declared
`[compat].bitty` floors are minimum-only except statusline, which has no
registry entry. Registry inclusion and a matching hash record integrity and
declared compatibility; they do not prove that a plugin is usable, shipped, or
verified, and they do not complete
[official schema issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

| Plugin (pin)            | `[settings]` declarations                                                          | Read behavior on the strict host                              |
| ----------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Activity `0ea39e13`     | `retention_days` (int 7), `store_command_args` (bool false)                        | Guarded; `store_command_args` stays inert either way          |
| Bar `7b6beb1f`          | 21 keys: six scalars, three module-group arrays, twelve empty module-option arrays | Scalars and groups apply; module option maps stay inert       |
| Composer `8be240f2`     | `newline`, `submit`, `close`, `external_editor`, `open_chord` (strings)            | All five declared and read                                    |
| Copy mode `2b78ceb3`    | `row_count` (int 10), `max_bytes` (int 4096)                                       | Declared and read                                             |
| Search `ee9a8f9c`       | `row_count` (int 10), `max_bytes` (int 4096)                                       | Declared and read                                             |
| Palette `5169d974`      | `query` (string `""`)                                                              | Declared; `entries` is no longer read                         |
| File manager `558bf5a2` | `root` (string `""`), `entries` (string-array `[]`), `width` (int 0)               | Declared and read; table-shaped `entries` is still unaccepted |
| Git panel `4f54634f`    | none                                                                               | No `bitty.settings` read                                      |
| DevTools `4e8e9680`     | none                                                                               | No `bitty.settings` read                                      |
| Statusline `2b9973c7`   | none                                                                               | Six guarded reads denied; local defaults serve                |

Statusline's repository is archived on GitHub (last push 2026-10-02) and has no
registry entry; its consolidated presentation is carried by the bar manifest's
own description, not by a settings contract. Statusline declarations therefore
have no active owner and must not be inferred from bar's declarations.

Wheel and AI behavior are not in this registry set. Their configuration work is
recorded as an open question below, not as part of this reference's schema.

## Open contract questions

These points are unresolved. None of them authorizes a broader settings kind,
a new file location, or a second loader in this reference.

- **Table-shaped `entries`.** `SettingKind` accepts only `bool`, `int`,
  `string`, and `string-array`, so a list of tables has no declaration form.
  File manager declares `entries` as `string-array` and uses it only when the
  host delivers a non-empty array; palette no longer reads the key. Accepting a
  table kind, a nested array, or a dot-path spelling is a manifest-schema
  decision owned by the Core `bitty-plugin-host` manifest owner together with
  [bitty-plugins#90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
- **Bar per-module option maps.** Bar declares twelve module option collections
  as empty `string-array` values while its Lua reads one table per module name.
  Map-shaped user options are inert on a strict host; the empty declaration is a
  fail-to-default posture, not an accepted option schema. Closing this needs a
  table or dot-path kind (or a different transport) decided by the bar owner and
  the manifest-schema owner.
- **Missing statusline declarations.** Six formatting keys
  (`separator`, `show_cwd`, `show_title`, `show_exit`, `max_components`,
  `component_max_chars`) are read but never declared. With the repository
  archived, no active owner can declare them; a successor surface must state
  the keys itself.
- **Enum-shaped closed sets.** Bar's `label` and `icons` are bounded closed sets
  validated in Lua because no enum kind exists; an unknown value falls back and
  emits one bounded diagnostic. Whether the manifest gains an enum kind is a
  Core manifest-schema question, not a documentation choice.
- **Wheel's direct project-config loader.** The `bitty-terminal.wheel` manifest
  declares no `[settings]`, and `lua/wheel/config.lua` resolves
  `$XDG_CONFIG_HOME/wheel/init.lua` and a project `.wheel/init.lua` outside
  `bitty.settings`, behind its own trust store. Core
  `crates/bitty-config/src/wheel.rs` only locates `.wheel/project.toml` and
  reads nothing. This is a second configuration path with its own consent gate;
  whether project configuration becomes a declared settings layer is owned by
  Wheel, with no change accepted here.

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
- Core `b5145675`: `crates/bitty-plugin-host/src/manifest.rs:1204-1212`
  (`MAX_SETTINGS_KEYS`, key and default bounds),
  `crates/bitty-runtime/src/plugin_runtime/services.rs:5070` (bridge key
  validation), `crates/bitty-config/src/plugin_file.rs:140-158` (legacy TOML
  bounds), `crates/bitty-config/src/wheel.rs` (project-directory discovery
  only).
- `bitty-plugins` `2da1c87`: `registry/official/*.toml` entries and the pinned
  plugin submodules whose manifests this page reconciles.
- [Official schema follow-up #90](https://github.com/bitty-terminal/bitty-plugins/issues/90)
- [Documentation follow-up #153](https://github.com/bitty-terminal/bitty-plugins-docs/issues/153)
- Additional inspected sources at the same `bitty-plugins` revision:
  statusline `2b9973c7` (archived repository), devtools `4e8e9680`,
  composer `8be240f2`, copy-mode `2b78ceb3`, and search `ee9a8f9c`.
