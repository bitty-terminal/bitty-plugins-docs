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

## Manifest

| Field                 | Value                      |
| --------------------- | -------------------------- |
| `[plugin].id`         | `bitty-terminal.git-panel` |
| `[plugin].name`       | Git Panel                  |
| `[plugin].version`    | 0.1.0                      |
| `[compat].bitty`      | `>=0.1,<1.0`               |
| `[compat].plugin-api` | `^1.0`                     |

## Capabilities

| Identifier               | Scope                                                                  |
| ------------------------ | ---------------------------------------------------------------------- |
| `panel.provider`         | Tiled Panel Runtime registration                                       |
| `panel.create`           | Panel creation within the host-owned panel slot                        |
| `terminal.semantic-read` | Read-only cwd/title observation via `bitty.terminal.snapshot`          |
| `process.spawn:git`      | Only the `git` binary, only the seven allowlisted read-only verbs      |
| `fs.read:~/projects/**`  | Working-tree read scope; symlinks and devices rejected per host policy |

Deny by default; unknown identifiers fail validation and there is no
allow-all identifier. No ambient spawn, shell interpolation, network,
clipboard, terminal-input, or persistent-state authority is requested.

## `[tools.git]` declaration

The accepted Layer 2 slice (`bitty` `CTX-0425`):

| Field      | Value    |
| ---------- | -------- |
| `required` | `true`   |
| `version`  | `>=2.30` |

Static, validated before VM creation, and included in the manifest hash for
grant binding. The canonical record is the accepted `[tools.git]` contract
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

## Authority

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
