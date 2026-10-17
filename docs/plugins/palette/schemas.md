---
title: Palette plugin schemas and contracts
description: Manifest surface version ranges capabilities commands and events for the palette plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 36
---

# Palette plugin schemas and contracts

## Purpose and scope

Describe the focusable overlay and configuration gap at registry-pinned
palette `c17dc321c8c7d4c8a1b8c5e7853d33e943af7c3d`.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns declarations and grants. The
[settings reference](../../../sdk/reference/settings.md) describes the current
declared-key boundary and restricted configuration loader.

## Manifest

| Field                 | Value                    |
| --------------------- | ------------------------ |
| `[plugin].id`         | `bitty-terminal.palette` |
| `[plugin].name`       | Palette                  |
| `[plugin].version`    | 0.0.1                    |
| `[compat].bitty`      | `>=0.0.24,<0.1`          |
| `[compat].plugin-api` | `^1.0`                   |

## Capabilities

`ui.overlay.focus` only. The source acquires a focusable overlay session with
`bitty.ui.overlay.acquire({ title, placeholder })`, updates the session-owned
surface, and releases capture. It does not use the non-focusable
`ui.rich`/`ui.overlay` slot mount path. Deny by default; release remains
available for cleanup when grants are revoked.

## Lazy surface

| Trigger  | Values                              |
| -------- | ----------------------------------- |
| Commands | `bitty-terminal.palette:toggle`     |
| Events   | `focus.changed`, `overlay.released` |

Static triggers let the host register the plugin without creating a VM.

## Settings gap and scene

The manifest has no `[settings]`; guarded `entries` and `query` reads receive
`E_SETTINGS_UNDECLARED` and fall back to `{}` and `""`. A settings file cannot
populate this palette yet. Table-shaped `entries` remains an unaccepted,
deferred question in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90), not a
new scalar/string-array declaration. The current scene is a `List` of `Text`
rows with title and empty placeholder. Live query typing, command-registry
enumeration, and a picker-provider API are not implemented by this source.

## Security review

The host owns transient input capture and its lifetime; a missing grant or
foreign capture degrades to command-only counts. No terminal-input injection,
filesystem, network, process, or clipboard authority is implied.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
