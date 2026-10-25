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

Describe the current registry-pinned bar `0fa151395fe9d9d8b3b9dc4ba047eb13fa7b45c4`
(`0fa15139`). The broader unified-chrome candidate does not define this pin's
implemented widgets or interactions.

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
| `[compat].bitty`      | `>=0.0.24`           |
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

| Key              | Kind           | Shipped default                           | Implemented behavior                                                  |
| ---------------- | -------------- | ----------------------------------------- | --------------------------------------------------------------------- |
| `show`           | `bool`         | `true`                                    | Enable presentation                                                   |
| `edge`           | `string`       | `"bottom"`                                | `"top"` or `"bottom"` semantic slot; other strings use local default  |
| `show_single`    | `bool`         | `false`                                   | Show a lone workspace; occupied scratchpad also keeps the bar visible |
| `name_max_chars` | `int`          | `32`                                      | Local validation permits 1 through `NAME_MAX_CHARS` (32)              |
| `label`          | `string`       | `"index"`                                 | Capsule label mode: `"index"`, `"name"`, or `"both"`                  |
| `modules_left`   | `string-array` | `["workspaces"]`                          | First ordered module group                                            |
| `modules_center` | `string-array` | `["clock"]`                               | Center group, centered across the full band                           |
| `modules_right`  | `string-array` | `["network", "cpu", "memory", "battery"]` | Right group, aligned to the right inset edge                          |

The shipped fresh-install plan places workspace capsules on the left, the clock
in the center, and provider-backed `network`, `cpu`, `memory`, and `battery`
modules on the right. Each unavailable or ungranted provider hides only its own
module. Consequently, a host without any of those providers may show only the
workspace capsules and clock; an individual provider can disappear while the
other right-side modules remain visible. Setting a group to `[]` removes that
group as long as another group still resolves to a known module. When the
configured plan contains no group, or no entry resolves to a known module, the
plugin falls back to the shipped default plan, so a bad or empty specification
never hides the bar.

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
Settings are resolved at activation and re-read on `config.reloaded`; only the
`edge` selection remains pinned to the generation.

Lazy commands, each under `bitty-terminal.bar:`, are `focus`, `detail`,
`bluetooth-toggle`, `backlight-step`, `idle-toggle`, `audio-toggle`, and
`audio-step`. Inactive workspace text binds the focus command, which requests
`bitty.workspace.focus`; provider controls use their separate grants.
Detail text can fall back to best-effort `bitty.notify.show`, subject to
`platform.notify` and host rate limits.

## Scene contract

The implementation mounts semantic `top` or `bottom` with `bitty.ui.mount`
under `ui.rich`, updates the generation-owned handle, and emits a root `Row`
containing only non-empty anchored groups. The left group is anchored to the
left inset, the center group is centered across the full band, and the right
group is aligned to the right inset; the provider-backed right group may be
absent. Hidden provider modules emit no atoms and are omitted individually.

Each workspace is one styled `Text` atom with one space of padding on each side
(`" 1 "`). The padding shares the capsule fill and stays inside the same atom,
so the whole chip, padding included, is one click target and reads as a filled
capsule rather than bracketed text. Marks append after the label in `*` (active),
`+` (occupied), and `!` (attention) order in every label mode. The active
capsule emits `bold` with the `background` foreground on the `accent`
background; inactive capsules use the `selection` background. On an inactive
capsule, occupied emphasis is bold and attention emphasis uses the `cursor`
foreground, with attention winning when both apply. Theme tokens are
host-resolved names, never literal colors. The v1 band exposes no corner-radius
primitive, so capsules are square padded fills and the plugin has no
rounded-corner option. Padding and fill apply to workspace capsules only; other
modules carry no capsule padding or fill and are joined by a single space.
A capsule remains a `Text` atom, not a distinct `Capsule` node; command bindings
decorate it. New widgets and add/close workspace buttons are not established by
this source.

## Security review

The plugin alters presentation only. Provider data and observations remain
untrusted, controls require separate consent, and notification acceptance is
an asynchronous hand-off rather than delivery confirmation. Settings cannot
create capabilities or bypass host UI budgets and generation ownership.

## Verification

The [evidence register](evidence.md) identifies the immutable registry and
source pins. This pass independently inspected manifest and Lua structure; it
does not claim a live UI campaign, new security acceptance, or release delivery.
