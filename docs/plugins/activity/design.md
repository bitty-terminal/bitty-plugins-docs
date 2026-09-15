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

## Problem

Users want a local record of what they worked on in the terminal without
sending command data anywhere.

## Scope

In scope: local, privacy-first aggregation of activity (counts, durations,
working directory) derived from read-only semantic terminal snapshots, and a
local summary surfaced through the plugin's declared commands.

Out of scope: storing command arguments or output, filesystem, process,
network, or clipboard authority, terminal input, and any remote telemetry.
Command arguments are never stored (`store_command_args` defaults to `false`).

## Capability and trust boundaries

- Requests only `terminal.semantic-read` (read-only committed-state
  observation) and `platform.notify`; the capability table is deny by default
  and has no allow-all identifier.
- Plugins alter presentation, never terminal truth; activity consumes
  observation events and does not enter the input, parser, or render hot paths.
- High-risk categories such as filesystem writes, process spawn, network,
  clipboard, terminal input, and agent/MCP calls are intentionally absent from
  the manifest.

## Failure behavior

Missing semantic zones or an absent host integration degrade the timeline
rather than failing activation. The plugin registers statically through lazy
triggers and creates no VM until a declared command or event fires; lifecycle
events flush pending aggregates before a generation is suspended or disposed.

## Alternatives considered

A server-side or synced timeline was rejected as incompatible with the
privacy-first position.

## Related

- [Activity plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
