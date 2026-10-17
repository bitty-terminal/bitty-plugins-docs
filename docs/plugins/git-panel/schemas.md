---
title: Git panel plugin schemas and contracts
description: Manifest surface tools git capabilities commands and events for the git panel plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 48
---

# Git panel plugin schemas and contracts

## Purpose and scope

Describe registry-pinned git-panel
`862df1f90ad137eb1e92379ac0710c6cd22c7f21`, its existing declarations, and
the limits of current Core integration. It has no settings reads or
`[settings]` declarations to reconcile.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns capability validation and grant binding. Only the linked accepted
`[tools.git]` slice below is accepted from the provider-ecology RFC.

## Manifest

| Field                 | Value                      |
| --------------------- | -------------------------- |
| `[plugin].id`         | `bitty-terminal.git-panel` |
| `[plugin].name`       | Git Panel                  |
| `[plugin].version`    | 0.1.0                      |
| `[compat].bitty`      | `>=0.0.24,<0.1`            |
| `[compat].plugin-api` | `^1.0`                     |

## Capabilities

| Identifier                  | Scope                                                                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `panel.provider`            | Tiled Panel Runtime registration — deferred candidate pending the panel-provider contract (`bitty-docs` `CTX-0181`, OQ-058)                |
| `panel.create`              | Panel creation within the host-owned panel slot — deferred candidate pending the panel-provider contract (`bitty-docs` `CTX-0181`, OQ-058) |
| `terminal.semantic-read`    | Read-only cwd/title observation via `bitty.terminal.snapshot`                                                                              |
| `process.spawn:git`         | Only the `git` binary, only the seven allowlisted read-only verbs                                                                          |
| filesystem read declaration | Manifest requests a working-tree pattern; real-path enforcement is not proved by the Lua policy tests                                      |

Deny by default; unknown identifiers fail validation and there is no
allow-all identifier. No ambient spawn, shell interpolation, network,
clipboard, terminal-input, or persistent-state authority is requested.

## `[tools.git]` declaration

The accepted Layer 2 slice (`bitty` `CTX-0425`):

| Field      | Value    |
| ---------- | -------- |
| `required` | `true`   |
| `version`  | `>=2.30` |

The accepted contract requires static validation before VM creation and
manifest-hash grant binding. Declaration or dispatch tests alone do not prove
the complete installation chain. The canonical record is the accepted `[tools.git]` contract
(v1) in the
[Plugin Reuse and Provider Ecology RFC](../../../packaging/plugin-reuse-and-providers.md#accepted-toolsgit-contract-v1);
the rest of that RFC stays draft.

## Lazy surface

| Trigger  | Values                                                                 |
| -------- | ---------------------------------------------------------------------- |
| Commands | `bitty-terminal.git-panel:open`, `:status`, `:diff`, `:log`, `:branch` |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`      |

Static triggers let the host register the plugin without creating a VM; the
observation events refresh the cached snapshot-derived state and never spawn.

## Policy bounds

Host-free Lua constants bound the composed listings and the spawn surface: at
most 128 status entries, 64 commits, a single branch bound of 32, names at 128
characters, commit messages at 256 characters, and at most 32 spawn args of at
most 256 bytes each.

## Current integration limits

Core `662212c5` has a real grant-gated `process.spawn:git` backend. Its Lua
bridge reads argv only; it does not forward the plugin's second
`{ cwd = cache.cwd }` argument. Repository isolation is therefore unproven,
and the plugin must not be advertised as a safe repository-aware panel.
This first-party process extra is outside the stable Plugin API v1 guarantee.

The plugin returns command data and composes `Text`/`List`/`Column` scenes;
it never mounts a panel. The requested panel capabilities do not implement a
Lua provider lifecycle. It does not call `bitty.fs` either.

## Security review

Bounded argv and read-only git verbs do not establish the intended cwd or
filesystem isolation. A future cwd contract needs host validation and real
backend tests before this gap can close. No schema or capability is widened
by this documentation update.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
