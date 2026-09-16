---
title: File manager plugin schemas and contracts
description: Manifest surface version ranges capabilities commands and events for the file manager plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 44
---

# File manager plugin schemas and contracts

## Manifest

| Field                 | Value                         |
| --------------------- | ----------------------------- |
| `[plugin].id`         | `bitty-terminal.file-manager` |
| `[plugin].name`       | File Manager                  |
| `[plugin].version`    | 0.1.0                         |
| `[compat].bitty`      | `>=0.1,<1.0`                  |
| `[compat].plugin-api` | `^1.0`                        |

## Capabilities

`terminal.semantic-read` only. Deny by default; unknown identifiers fail
validation and there is no allow-all identifier. No `panel.*`, `fs.*`,
process, network, clipboard, or terminal-input authority is requested.

## Lazy surface

| Trigger  | Values                                                            |
| -------- | ----------------------------------------------------------------- |
| Commands | `bitty-terminal.file-manager:open`, `:preview`, `:rename`         |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed` |

Static triggers let the host register the plugin without creating a VM; the
observation events refresh the cached snapshot-derived state.

## Configuration keys

The plugin reads two optional settings through `bitty.settings.get`, falling
back silently when the settings surface is absent:

| Key       | Type           | Purpose                                                                 |
| --------- | -------------- | ----------------------------------------------------------------------- |
| `root`    | string         | Scope root candidate, used when the command carries no explicit `root`. |
| `entries` | table of paths | Listing candidates, used when the command carries no explicit `paths`.  |

Both are inputs to the root-parameterized, fail-closed scope; neither grants
authority beyond `terminal.semantic-read`.

## Policy bounds

Host-free Lua constants bound the composed listings: at most 128 entries per
listing, names at 128 code points, paths at 4096 bytes, selection at 64, and
the total listing payload at 8192 bytes (`8 KiB`).

## Authority

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
