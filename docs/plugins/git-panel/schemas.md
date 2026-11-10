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

Describe registry-pinned git-panel `4f54634f` (manifest
`sha256:fb18021aba54da25256d01edf2e6fdc6caef9012e9e1f71f1af74ac745f2ace6`), its
declarations, and the limits of current Core integration. It has no settings
reads or `[settings]` declarations to reconcile.

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
| `[compat].bitty`      | `>=0.0.24`                 |
| `[compat].plugin-api` | `^1.0`                     |

## Capabilities

| Identifier                  | Scope                                                                                                                                          |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `ui.overlay.focus`          | The accepted W-01 coupled grant: acquire the focusable overlay, hold its transient input-capture session, and release it on `overlay.released` |
| `terminal.semantic-read`    | Read-only cwd/title observation via `bitty.terminal.snapshot`                                                                                  |
| `process.spawn:git`         | Only the `git` binary, only the seven allowlisted read-only verbs, with `cwd = "pane"`                                                         |
| filesystem read declaration | `access = "read"`, `paths = ["**"]`; the host anchors the pattern at the directory a call requests                                             |

Deny by default; unknown identifiers fail validation and there is no
allow-all identifier. No `panel.provider` or `panel.create` capability is
requested: the panel surface is the focusable overlay session, and
`bitty.panel.create` only allocates an inert host-owned record. No ambient
spawn, shell interpolation, network, clipboard, terminal-input, or
persistent-state authority is requested.

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

| Trigger  | Values                                                                                |
| -------- | ------------------------------------------------------------------------------------- |
| Commands | `bitty-terminal.git-panel:open`, `:status`, `:diff`, `:log`, `:branch`                |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`, `overlay.released` |

Static triggers let the host register the plugin without creating a VM; the
observation events refresh the cached snapshot-derived state and never spawn.
`overlay.released` clears the local overlay session when Core ends it.

## Policy bounds

Host-free Lua constants bound the composed listings and the spawn surface: at
most 128 status entries, 64 commits, a single branch bound of 32, names at 128
characters, commit messages at 256 characters, and at most 32 spawn args of at
most 256 bytes each.

## Current integration limits

The pinned source presents through the focusable overlay
(`bitty.ui.overlay.acquire` / `update`) rather than a host panel slot, and
releases the session on `overlay.released`. Panel usability is not proven by
that composition alone: no live panel campaign backs this page.

Spawn passes the host-resolved semantic `cwd = "pane"`; the plugin never
supplies a path. Core resolves the globally focused view's PTY-child kernel
directory at call time, validates it, records it in the audit entry, and fails
closed as `Denied[SpawnCwdUnavailable]` when no binding or provable kernel cwd
exists. That cwd is a starting directory, not a sandbox: `process.spawn:git` is
itself a repository-read capability, so the child reads the pane tree with the
user's OS authority, and `.git`/`core.worktree` metadata remains a separate
channel with a tracked follow-up. Repository isolation is therefore a host
property this page does not independently verify.
This first-party process extra is outside the stable Plugin API v1 guarantee.

## Security review

Bounded argv, the read-only git verbs, and the host-resolved `pane` cwd are
separate controls. The host validates and audits the resolved directory and
fails closed without a provable kernel cwd; it does not sandbox the child, which
still runs with the user's OS authority and can follow repository metadata. A
future `.git` policy and real-backend isolation tests are required before this
gap can close. No schema or capability is widened by this documentation update.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
