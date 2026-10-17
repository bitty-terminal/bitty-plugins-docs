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

## Purpose and scope

Describe statusline at inspected source
`2b9973c7569398437ba632b41cce1fa37577563c`. It is absent from registry #92's
five eligible pins and remains pending; this is not Core 0.0.24 eligibility
or end-to-end verification evidence.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and declarations. The
[settings reference](../../../sdk/reference/settings.md) describes the
strict declared-key gate.

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

## Settings gap

The inspected manifest has no `[settings]`. The source attempts guarded reads
for `separator` (string), `show_cwd`, `show_title`, `show_exit` (booleans),
and `max_components`, `component_max_chars` (integer candidates). On the
strict host these reads are undeclared and local formatting defaults serve.
These types are reconciliation candidates, not declarations made by this page.
Follow-up remains in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

## Security review

Formatting remains presentation-only and never alters terminal truth or
grants. The source composes `Row`/`Text`; this does not create new widgets or
general panel authority.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
