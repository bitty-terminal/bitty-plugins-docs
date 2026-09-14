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

## Manifest

| Field                 | Value                     |
| --------------------- | ------------------------- |
| `[plugin].id`         | `bitty-featured.activity` |
| `[plugin].name`       | Bitty Activity            |
| `[plugin].version`    | 0.1.0                     |
| `[compat].bitty`      | `>=0.5,<1.0`              |
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

## Authority

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
