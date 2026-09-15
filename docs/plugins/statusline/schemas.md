---
title: Statusline plugin schemas and contracts
description: Manifest surface version ranges capabilities and events for the statusline plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 40
---

# Statusline plugin schemas and contracts

## Manifest

| Field                 | Value                       |
| --------------------- | --------------------------- |
| `[plugin].id`         | `bitty-terminal.statusline` |
| `[plugin].name`       | Statusline                  |
| `[plugin].version`    | 0.0.1                       |
| `[compat].bitty`      | `>=0.1,<1.0`                |
| `[compat].plugin-api` | `^1.0`                      |

## Capabilities

`terminal.semantic-read`, `ui.rich`. Deny by default; unknown identifiers fail
validation and there is no allow-all identifier.

## Lazy surface

| Trigger  | Values                                                                               |
| -------- | ------------------------------------------------------------------------------------ |
| Commands | none                                                                                 |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`, `terminal.opened` |

Static triggers let the host register the plugin without creating a VM; the
observation events activate and drive recomposition.

## Authority

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
