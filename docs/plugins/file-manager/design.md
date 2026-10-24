---
title: File manager plugin design
description: Scope observation-only policy and capability design for the file manager plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 43
---

# File manager plugin design

## Document status

Observation-only source at registry-pinned `bf3a17ea`; filesystem access and
panel presentation remain deferred. No end-to-end file-manager claim is made.

## Purpose and scope

Users need file listing, navigation, and preview driven by the observed
terminal working directory without leaving the terminal workspace.

## Normative sources this specification must not weaken

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and isolation. The
[settings reference](../../../sdk/reference/settings.md) describes the strict
declared-key filter.

## Terminology

A listing here shapes supplied candidate paths; it does not enumerate host
directories. Preview is path-derived data, not a file-content read.

## Current behavior

In scope: observation-only listing, navigation, and preview policy over the
observed terminal cwd, with a root-parameterized, fail-closed scope and a
bounded `8 KiB` listing payload. The Lua policy never touches the filesystem
directly; it composes paths against a caller-supplied root and degrades to an
empty listing when no root is available.

Out of scope: terminal truth, input, parser, and render hot paths; panel
presentation; and direct filesystem I/O. Plugin API v1 has no panel-mount or
filesystem surface (`bitty.ui.register_panel` is post-v1.0 and there is no
`bitty.fs`), so the package cannot register a panel provider until a future
panel-provider acceptance lands. These deferred surfaces are explicit
non-goals of the current package, not implied behavior.

## Mechanism and policy split

Core (host) owns terminal truth, semantic snapshots, and — once the deferred
contracts land — panel mounting and filesystem mediation. The Lua package owns
the presentation policy: root resolution (explicit arg, then an attempted
`root` setting read, then the cached snapshot cwd), bounded listing composition
(`MAX_ENTRIES` 128, `PAYLOAD_MAX_BYTES` 8192), filtering, sorting, and preview
selection. The scene module is an intentional placeholder for the deferred
panel presentation.

The manifest declares no settings. Guarded `root`/`entries` reads are denied
by the strict host, so explicit arguments, cached cwd, or an empty listing
serve. Table-shaped `entries` remains unaccepted and deferred in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).

## Failure behavior

The plugin registers statically through lazy triggers and creates no VM until
a command or observation event fires. A stale or denied snapshot never aborts
the operation; missing semantic zones degrade the cached state rather than
failing activation. Panel absence leaves no file-manager surface.

## Security review

- Requests `terminal.semantic-read` only (gates `bitty.terminal.snapshot`,
  the read-only cwd/title observation the cache refreshes from). This is the
  single capability.
- The former bundled `panel.provider`, `panel.create`, and root-scoped
  `fs.read`/`fs.write` requests are removed as phantom authority: no Lua here
  calls a panel API, and no accepted Plugin API v1 surface exposes
  `bitty.fs`. A grant is re-added only when code actually exercises it.
- Deny by default: no process, network, clipboard, terminal-input, or
  persistent-state authority is requested, and no install-time code runs.

## Verification plan

Verify path shaping, root containment, bounds, guarded undeclared reads, and
absence of filesystem/panel calls at the pinned source. Host real-path and
mutation tests are required before future I/O can be claimed.

## Alternatives considered

A bundled-only file-manager was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved it to an independent first-party package with the catalog entry
removed and panel presentation deferred.

## Affected contracts

No filesystem grant, panel-provider API, or table-valued settings kind is
accepted by this documentation. `rename` still returns a validated pair only.

## Open points

Settings declarations and entry shape remain issue #90. Host filesystem
backend, mutation authority, and provider lifecycle remain separate work.

## Acceptance criteria

Keep observation-only behavior explicit until accepted contracts, production
backends, and independent tests establish real filesystem/panel behavior.

## P0 Review Sign-off

No new security acceptance is claimed by this source synchronization.

## References

- [File manager plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
