---
title: DevTools plugin schemas and contracts
description: Manifest surface capabilities commands trace events and absent settings for the devtools plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 59
---

# DevTools plugin schemas and contracts

## Purpose and scope

Describe registry-pinned DevTools `4e8e9680` (manifest
`sha256:a8c4755f02254e0f8c085755279f5b25951e55b02423730afe0f85ae02678f26`). The
page covers the read-only inspection and tracing surface, and states explicitly
that the package declares no settings. The candidate GUI live view is a separate
draft direction, not behavior of this pin.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) owns
declarations, grants, and lifecycle. The
[settings reference](../../../sdk/reference/settings.md) defines the
declared-key gate that an empty `[settings]` table produces.

## Manifest

| Field                 | Value                     |
| --------------------- | ------------------------- |
| `[plugin].id`         | `bitty-featured.devtools` |
| `[plugin].name`       | Bitty DevTools            |
| `[plugin].version`    | 0.1.0                     |
| `[compat].bitty`      | `>=0.0.21`                |
| `[compat].plugin-api` | `^1.0`                    |

The Bitty range is minimum-only, so a newer host is admitted instead of being
rejected by a ceiling. The floor is the first host carrying the surfaces the
plugin uses; it is not a verified release claim.

## Capabilities

| Identifier        | Scope                                                                          |
| ----------------- | ------------------------------------------------------------------------------ |
| `debug.inspect`   | Read-only `bitty.debug.inspect`: lifecycle rows, registrations, and own grants |
| `debug.trace`     | Bounded `bitty.debug.trace` / `trace_get` event traces                         |
| `platform.notify` | Bounded text summaries                                                         |

`debug.control` and every `ui.*` identifier are deliberately absent: the plugin
never mutates runtime state and its UI is notification-only. Deny by default;
unknown identifiers fail validation and there is no allow-all identifier.

## Commands and trace events

Lazy commands, each under `bitty-featured.devtools:`, are `plugins`, `commands`,
`events`, `grants`, `trace-start`, `trace-stop`, and `trace-dump`. The bridge
keeps `id`, `title`, `description`, and `run`; `args_schema`/`result_schema` are
documentation only, so `trace-start` validates its own `filter` argument.

The declared events are `terminal.opened`, `terminal.closed`,
`terminal.title-changed`, `terminal.cwd-changed`, `terminal.bell`,
`focus.changed`, `selection.changed`, `process.exited`, and `config.reloaded`.
Declaring a kind is what makes it visible to a trace; the plugin subscribes to
none of them, and only v1 observation kinds are traceable.

## Settings

The manifest has no `[settings]` table and the source performs no
`bitty.settings.get` read, so the generation's declared-key set is empty and any
settings read would fail closed with `E_SETTINGS_UNDECLARED`. Configuration
cannot change what the plugin inspects, which grants it sees, or how much trace
data it retains. A future option would need a declaration plus a host bound; this
page declares none.

## Security review

Every host call is wrapped so a missing bridge or revoked grant degrades to a
bounded error result rather than a raise. Inspected runtime data stays untrusted
input to formatting, notifications carry bounded text only, and the plugin holds
no filesystem, process, network, clipboard, terminal-input, or runtime-control
authority. Trace retention is bounded host-side; the plugin adds no store of its
own.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. This page records source and manifest inspection only;
no live tracing campaign was run.
