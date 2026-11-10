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

Describe the focusable overlay and the declared `query` setting at
registry-pinned palette `5169d974` (manifest
`sha256:53e334de7f98631f68af21495d1dc9c40c7ed27211cdab5062ed0c95ef191547`).

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
| `[compat].bitty`      | `>=0.0.24`               |
| `[compat].plugin-api` | `^1.0`                   |

## Capabilities

`ui.overlay.focus` only. The source acquires a focusable overlay session with
`bitty.ui.overlay.acquire({ title, placeholder })`, updates the session-owned
surface, and releases capture. It does not use the non-focusable
`ui.rich`/`ui.overlay` slot mount path. Deny by default; release remains
available for cleanup when grants are revoked.

## Lazy surface

| Trigger  | Values                                                         |
| -------- | -------------------------------------------------------------- |
| Commands | `bitty-terminal.palette:toggle`                                |
| Events   | `focus.changed`, `overlay.released`, `overlay.input.available` |

Static triggers let the host register the plugin without creating a VM.
`overlay.input.available` is an observation-only nudge: while this plugin owns
the capture session it drains queued keys through `bitty.ui.overlay.poll`.

## Settings and scene

The manifest declares one key, `query` (`string`, default `""`): the filter
seeded when the overlay opens. The activation read is truncated to
`filter.MAX_QUERY_CHARS` (128 characters) and stripped of control characters;
live typing then replaces it for the session, so the setting is the
activation-time seed rather than a live filter. `entries` is not read by this
source at all; the list comes from the host command catalog.

A table-shaped `entries` kind remains unaccepted and deferred in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90), not a
scalar or string-array declaration this page invents. The current scene is a
`List` of `Text` rows with title and empty placeholder. Command-registry
enumeration and a picker-provider API are not implemented by this source.

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
