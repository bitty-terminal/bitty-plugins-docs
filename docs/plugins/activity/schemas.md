---
title: Activity plugin schemas and contracts
description: Manifest surface version ranges capabilities commands and events for the activity plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 32
---

# Activity plugin schemas and contracts

## Purpose and scope

Describe the manifest and settings behavior at registry-pinned activity
`e1566380b9bddb92a5db8b9f1ba96c080995ef8a`. Registry #92 admits the package
for Core 0.0.24; it does not complete the settings declarations.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns manifest validation and grants; the
[settings reference](../../../sdk/reference/settings.md) describes the current
declared-key gate and restricted configuration loader.

## Manifest

| Field                 | Value                     |
| --------------------- | ------------------------- |
| `[plugin].id`         | `bitty-featured.activity` |
| `[plugin].name`       | Bitty Activity            |
| `[plugin].version`    | 0.0.1                     |
| `[compat].bitty`      | `>=0.0.24,<0.1`           |
| `[compat].plugin-api` | `^1.0`                    |

## Capabilities

`terminal.semantic-read`, `platform.notify`. Deny by default; no allow-all
identifier exists. Requested identifiers are validated against the accepted
registry and unknown values fail validation.

## Lazy surface

| Trigger  | Values                                                                                                                |
| -------- | --------------------------------------------------------------------------------------------------------------------- |
| Commands | `bitty-featured.activity:summary`, `bitty-featured.activity:clear`                                                    |
| Events   | `terminal.opened`, `terminal.closed`, `terminal.cwd-changed`, `process.exited`, `plugin.suspended`, `plugin.disposed` |

Static triggers let the host register the plugin without creating a VM; the
lifecycle events flush pending aggregates before suspension or disposal.

## Settings gap

The pinned manifest has no `[settings]`. `lua/activity/init.lua` reads
`retention_days` and `store_command_args` inside `pcall`; the strict host
returns `E_SETTINGS_UNDECLARED`, so local defaults serve (seven days and
`false`). A file cannot make these undeclared overrides effective.
`store_command_args = true` would still not store command arguments in this
implementation. Declaration work remains in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

## Security review

Settings do not grant authority. Semantic observations remain untrusted, and
notifications require `platform.notify` plus host rate policy. No filesystem,
process, or network capability is implied by local aggregation.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.

The [evidence register](evidence.md) records the inspected pin and remaining
schema work; this documentation pass does not rerun plugin product tests.
