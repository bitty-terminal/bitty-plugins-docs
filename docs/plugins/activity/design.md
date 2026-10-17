---
title: Activity plugin design
description: Scope capability privacy and failure design for the activity timeline plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 31
---

# Activity plugin design

## Document status

Current source at registry-pinned `e1566380`, without release verification.
Manifest settings reconciliation remains open in issue #90.

## Purpose and scope

Users want a local record of what they worked on in the terminal without
sending command data anywhere.

## Normative sources this specification must not weaken

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns declarations, grants, and generation lifetime. The
[settings reference](../../../sdk/reference/settings.md) describes the current
restricted loader and read gate.

## Terminology

An activity aggregate records counts, durations, and cwd observations; it is
not a command transcript or argument store.

## Current behavior

In scope: local, privacy-first aggregation of activity (counts, durations,
working directory) derived from read-only semantic terminal snapshots, and a
local summary surfaced through the plugin's declared commands.

Out of scope: storing command arguments or output, filesystem, process,
network, or clipboard authority, terminal input, and any remote telemetry.
Command arguments are never stored (`store_command_args` defaults to `false`).

## Failure behavior

Missing semantic zones or an absent host integration degrade the timeline
rather than failing activation. The plugin registers statically through lazy
triggers and creates no VM until a declared command or event fires; lifecycle
events flush pending aggregates before a generation is suspended or disposed.

## Security review

- Requests only `terminal.semantic-read` (read-only committed-state
  observation) and `platform.notify`; the capability table is deny by default
  and has no allow-all identifier.
- Plugins alter presentation, never terminal truth; activity consumes
  observation events and does not enter the input, parser, or render hot paths.
- High-risk categories such as filesystem writes, process spawn, network,
  clipboard, terminal input, and agent/MCP calls are intentionally absent from
  the manifest.

## Verification plan

Inspect guarded reads, argument-storage exclusion, bounded aggregation, store
failure recovery, and notification denial. Plugin test presence does not prove
live integration or release delivery.

## Alternatives considered

A server-side or synced timeline was rejected as incompatible with the
privacy-first position.

## Affected contracts

`retention_days` and `store_command_args` are undeclared in this pin. Guarded
reads use seven-day and false local defaults. Argument storage stays absent
regardless of the latter value; configuration cannot widen authority.

## Open points

[Issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90) owns the
missing declarations. This page does not add them by documentation.

## Acceptance criteria

Effective settings require matching manifest declarations and source behavior.
End-to-end claims require independent integration evidence.

## P0 Review Sign-off

No new security acceptance is claimed by this source synchronization.

## References

- [Activity plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
