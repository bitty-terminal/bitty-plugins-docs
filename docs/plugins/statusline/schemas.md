---
title: Statusline plugin schemas and contracts
description: Archived statusline manifest surface version ranges capabilities events and undeclared settings
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 40
---

# Statusline plugin schemas and contracts

## Purpose and scope

Describe the archived statusline source at `2b9973c7`. The repository is
archived on GitHub (last push 2026-10-02) and has no entry under
`registry/official/` in `bitty-plugins`, so this page is neither eligibility
evidence nor a claim that a current host can install the package. It records
what the archived manifest and Lua actually contain.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and declarations. The
[settings reference](../../../sdk/reference/settings.md) describes the
strict declared-key gate and the restricted configuration loader.

## Manifest

| Field                 | Value                       |
| --------------------- | --------------------------- |
| `[plugin].id`         | `bitty-terminal.statusline` |
| `[plugin].name`       | Statusline                  |
| `[plugin].version`    | 0.0.1                       |
| `[compat].bitty`      | `>=0.1,<1.0`                |
| `[compat].plugin-api` | `^1.0`                      |

The compatibility range is the only ceiling-declaring range among the audited
plugin pages; the other official manifests are minimum-only. A range alone
admits nothing: the repository is archived and no registry entry publishes it.

## Capabilities

`terminal.semantic-read` and `ui.rich`. Deny by default; unknown identifiers
fail validation and there is no allow-all identifier. `terminal.semantic-read`
gates the read-only committed-state observation the statusline composes from,
and `ui.rich` gates `bitty.ui.mount`/`bitty.ui.update` on the `statusline` slot.

## Lazy surface

| Trigger  | Values                                                                               |
| -------- | ------------------------------------------------------------------------------------ |
| Commands | none                                                                                 |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`, `terminal.opened` |

Static triggers let the host register the plugin without creating a VM; the
observation events activate and drive recomposition.

## Settings gap

The archived manifest has no `[settings]` table. `lua/statusline/init.lua`
attempts guarded reads for six keys through `bitty.settings.get`:

| Key                   | Read as | Local default                     |
| --------------------- | ------- | --------------------------------- |
| `separator`           | string  | the module's own separator        |
| `show_cwd`            | boolean | enabled                           |
| `show_title`          | boolean | enabled                           |
| `show_exit`           | boolean | enabled                           |
| `max_components`      | number  | the formatter's component ceiling |
| `component_max_chars` | number  | the formatter's per-component cap |

On a strict host every read is undeclared, so `E_SETTINGS_UNDECLARED` is caught
and the local defaults serve. These types are reconciliation candidates that the
archived manifest never declared; this page does not declare them either.
Because the repository is archived, the declarations have no active owner — a
successor surface must state its own keys. The open question is recorded in the
[settings reference](../../../sdk/reference/settings.md#open-contract-questions)
and in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

## Security review

Formatting remains presentation-only and never alters terminal truth or grants.
The source composes `Row`/`Text` under `ui.rich` and reads only the semantic
snapshot; no filesystem, process, network, clipboard, or terminal-input
authority is requested or implied by a formatting value.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. This page was reconciled against the archived
repository and the current registry index; no live statusline campaign was run.
