---
title: Bar plugin schemas and contracts
description: Current bar manifest settings scene and deferred option-map contracts
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 52
---

# Bar plugin schemas and contracts

## Purpose and scope

Describe registry-pinned bar `18d1a5b6dc86af4a42546f20f1610e69af58071d`.
The broader unified-chrome candidate does not define this pin's implemented
widgets or interactions.

## Normative sources

- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md):
  declarations, grants, and lifecycle.
- [Settings reference](../../../sdk/reference/settings.md): restricted
  configuration VM, strict kinds, precedence, and typed denials.
- [ADR 0014](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0014-workspace-core-presentation-plugins.md):
  Core owns workspace mechanism; plugins own presentation.

## Manifest

| Field                 | Value                |
| --------------------- | -------------------- |
| `[plugin].id`         | `bitty-terminal.bar` |
| `[plugin].name`       | Unified Bar          |
| `[plugin].version`    | `0.0.1`              |
| `[compat].bitty`      | `>=0.0.24,<0.1`      |
| `[compat].plugin-api` | `^1.0`               |

## Capabilities and providers

The pinned manifest requests `ui.rich`, `workspace.read`, `workspace.control`,
and `platform.notify`. Separate provider heads request
`platform.cpu-read`, `platform.memory-read`, `platform.battery-read`,
`platform.network.read`, `platform.essid.read`,
`platform.bluetooth.read`/`.control`, `platform.backlight.read`/`.control`,
`platform.idle.read`/`.control`, `audio.state.read`/`.control`, and
`audio.spectrum`. These declarations are requests, not automatic grants.
Read access never implies control. Missing or ungranted optional providers
hide their modules.

## Declared settings

Use `$XDG_CONFIG_HOME/bitty/plugins/bitty-terminal.bar.lua`, or the
unambiguous `bar.lua` alias, as described in the settings reference, not an
invented `[plugins]` table in `bitty.toml`. The legacy
`bitty-terminal.bar.toml` file remains a compatibility fallback only when no
selected Lua file exists.

| Key              | Kind           | Shipped default  | Implemented behavior                                                  |
| ---------------- | -------------- | ---------------- | --------------------------------------------------------------------- |
| `show`           | `bool`         | `true`           | Enable presentation                                                   |
| `edge`           | `string`       | `"bottom"`       | `"top"` or `"bottom"` semantic slot; other strings use local default  |
| `show_single`    | `bool`         | `false`          | Show a lone workspace; occupied scratchpad also keeps the bar visible |
| `name_max_chars` | `int`          | `32`             | Local validation permits 1 through `NAME_MAX_CHARS` (32)              |
| `label`          | `string`       | `"index"`        | Capsule label mode: `"index"`, `"name"`, or `"both"`                  |
| `modules_left`   | `string-array` | `["workspaces"]` | First ordered module group                                            |
| `modules_center` | `string-array` | `["clock"]`      | Second ordered group, without true centering                          |
| `modules_right`  | `string-array` | `[]`             | Third ordered group, without right alignment                          |

The shipped default plan is workspaces on the left, the clock in the center
group, and an empty right group. A fresh install therefore paints the workspace
capsules followed by the clock; setting a group to `[]` removes that group as
long as another group still resolves to a known module. When the configured plan
contains no group, or no entry resolves to a known module, the plugin falls back
to the shipped default plan, so a bad or empty specification never hides the
bar.

`label` is a bounded closed set with no v1 enum kind: the shipped default is
`index`, an unknown or non-string value falls back to `index` (the bar still
paints) and reports one bounded notification through the already-requested
`platform.notify` grant. `name_max_chars` only applies in `name` and `both`
modes, where an oversized name is truncated to that many characters.

`workspaces`, `clock`, `network`, `bluetooth`, `backlight`, `idle`,
`pulseaudio`, `spectrum`, `cpu`, `memory`, and `battery` are also declared,
each as `string-array` with empty default. Lua reads them as per-module
option collections, but strict settings kinds reject map-shaped values.
The empty arrays leave module defaults in effect. These declarations do not
accept maps or dot-path keys such as `workspaces.format`. That mismatch stays
explicit in [issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

## Events and commands

Lazy events are `workspace.created`, `workspace.closed`, `workspace.renamed`,
`workspace.focused`, `workspace.changed`, and `config.reloaded`.
The plugin reads `bitty.workspace.list()` on the configured event path.
Settings are resolved at activation and pinned to the generation, so
`config.reloaded` does not refresh them and no hot-reload path exists; a later
activation reads the configured file again.

Lazy commands, each under `bitty-terminal.bar:`, are `focus`, `detail`,
`bluetooth-toggle`, `backlight-step`, `idle-toggle`, `audio-toggle`, and
`audio-step`. Inactive workspace text binds the focus command, which requests
`bitty.workspace.focus`; provider controls use their separate grants.
Detail text can fall back to best-effort `bitty.notify.show`, subject to
`platform.notify` and host rate limits.

## Scene contract

The implementation mounts semantic `top` or `bottom` with `bitty.ui.mount`
under `ui.rich`, updates the generation-owned handle, and emits a `Row` of
`Text` nodes. Hidden content is empty `Text`. Bold/color and command bindings
decorate those text nodes. Bracketed workspace labels are text, not a
`Capsule` node. True three-zone alignment, new widgets, add/close workspace
buttons, and workspace wheel cycling are not established by this source.

## Security review

The plugin alters presentation only. Provider data and observations remain
untrusted, controls require separate consent, and notification acceptance is
an asynchronous hand-off rather than delivery confirmation. Settings cannot
create capabilities or bypass host UI budgets and generation ownership.

## Verification

The [evidence register](evidence.md) identifies the immutable registry and
source pins. This pass independently inspected manifest and Lua structure; it
does not claim a live UI campaign, new security acceptance, or release delivery.
