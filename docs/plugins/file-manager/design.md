---
title: File manager plugin design
description: Scope read-only listing policy and capability design for the file manager plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 43
---

# File manager plugin design

## Document status

Read-only listing source at registry-pinned `558bf5a2`. Enumeration is
host-mediated through `bitty.fs.list` under a read-only `fs.read:**` scope, and
the listing renders as a `bottom` band block. Panel presentation and real-path
isolation evidence remain deferred, so no end-to-end file-manager claim is made.

## Purpose and scope

Users need file listing, navigation, and preview driven by the observed
terminal working directory without leaving the terminal workspace.

## Normative sources this specification must not weaken

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and isolation. The
[settings reference](../../../sdk/reference/settings.md) describes the strict
declared-key filter.

## Terminology

A listing here is composed from a caller-chosen root; enumeration is
host-mediated and bounded, never raw directory I/O from Lua. Preview is
path-derived data, not a file-content read.

## Current behavior

In scope: read-only listing, navigation, and preview policy over the observed
terminal cwd, with a root-parameterized, fail-closed scope and a bounded `8 KiB`
listing payload. The Lua policy never opens a directory itself: it calls
`bitty.fs.list` under the declared read-only `fs.read:**` scope and degrades to
an empty listing when the host surface is absent or denied.

Out of scope: terminal truth, input, parser, and render hot paths; panel
presentation; and any write or mutation. `bitty.fs` is the Core RFC-0005
bridge beyond Plugin API v1, and `bitty.ui.register_panel` is post-v1.0, so the
package presents a `bottom` band block instead of registering a panel provider.
These deferred surfaces are explicit non-goals of the current package, not
implied behavior.

## Mechanism and policy split

Core (host) owns terminal truth, semantic snapshots, and filesystem mediation;
the `fs.read:**` grant is enforced per prefix, including sensitive-path and
secret gates. The Lua package owns the presentation policy: root resolution
(explicit arg, then the `root` setting, then the cached snapshot cwd), bounded
listing composition (`MAX_ENTRIES` 128, `PAYLOAD_MAX_BYTES` 8192), filtering,
sorting, and preview selection.

The manifest declares `root` (string), `entries` (string-array), and `width`
(int). `entries` is a string array only; a table-shaped list remains unaccepted
and deferred in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

## Failure behavior

The plugin registers statically through lazy triggers and creates no VM until
a command or observation event fires. A stale or denied snapshot never aborts
the operation; missing semantic zones degrade the cached state rather than
failing activation. A denied or absent `bitty.fs.list` degrades to an empty
listing, and a failed band mount leaves the previous scene in place.

## Security review

- Requests `terminal.semantic-read` (gates `bitty.terminal.snapshot`, the
  read-only cwd/title observation the cache refreshes from), `ui.rich` (gates
  the `bottom` band mount), and one read-only `fs.read:**` filesystem scope for
  `bitty.fs.list`.
- The former bundled `panel.provider`, `panel.create`, and `fs.write` requests
  stay removed: no Lua here calls a panel API or mutates a file, and the
  filesystem grant is read-only with a rootless pattern the host anchors at the
  requested directory. A grant is re-added only when code actually exercises it.
- Deny by default: no process, network, clipboard, terminal-input, or
  persistent-state authority is requested, and no install-time code runs.

## Verification plan

Verify root resolution and precedence, bounds, declared-key behavior, the
host-mediated `bitty.fs.list` call path, and the absence of panel and write
calls at the pinned source. Host real-path and mutation tests are required
before any isolation or write claim can be made.

## Alternatives considered

A bundled-only file-manager was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved it to an independent first-party package with the catalog entry
removed and panel presentation deferred.

## Affected contracts

The read-only `fs.read:**` scope and the three declared settings keys are
accepted as manifest declarations here; no panel-provider API and no
table-valued settings kind is accepted by this documentation. `rename` still
returns a validated pair only.

## Open points

The table-shaped `entries` shape remains issue #90. Real-path isolation
evidence, mutation authority, and provider lifecycle remain separate work.

## Acceptance criteria

Keep read-only behavior explicit until accepted contracts, real-path isolation
evidence, and independent tests establish panel and mutation behavior.

## P0 Review Sign-off

No new security acceptance is claimed by this source synchronization.

## References

- [File manager plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
