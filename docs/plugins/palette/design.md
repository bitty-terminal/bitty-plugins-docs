---
title: Palette plugin design
description: Scope overlay presentation and capability design for the palette plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 35
---

# Palette plugin design

## Document status

Current source at registry-pinned `c17dc321`; settings population and live
query typing remain incomplete. Registry eligibility is not product verification.

## Purpose and scope

Users need a keyboard-driven command and picker surface without leaving the
terminal workspace.

## Normative sources this specification must not weaken

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and lifetime. The
[settings reference](../../../sdk/reference/settings.md) describes the current
strict key/kind boundary.

## Terminology

The focusable overlay session owns transient capture and a host surface. It
is distinct from a non-focusable `ui.overlay` slot mount.

## Current behavior

In scope: an overlay presenting declarative lists and text primitives driven by
the plugin's toggle command and focus changes.

The source calls `bitty.ui.overlay.acquire({ title, placeholder })`, updates
the session-owned scene, and releases capture on close. `overlay.released`
resets local toggle state. The scene is a `List` of `Text` rows, including
title and empty placeholder. It requests `ui.overlay.focus` only.

Guarded `entries` and `query` reads are undeclared in the manifest and use
empty defaults on the strict host. A configuration file cannot populate the
list yet. Live typing, registry enumeration, picker providers, and terminal
truth modification are outside this source's implemented behavior.

## Failure behavior

The plugin registers through lazy triggers. Missing/denied overlay capture
degrades to command-only counts; a rejected update releases a failed session.
An external `overlay.released` resets toggle state so the next toggle can open.

## Security review

- Requests `ui.overlay.focus`; capture and surface lifetime remain host-owned.
- Missing/denied capture, a foreign session, or rejected update fails closed.
  Cleanup release stays available after grant revocation.
- Deny by default: no allow-all identifier, and no filesystem, process,
  network, clipboard, or terminal-input authority is requested.

## Verification plan

Verify acquire/update/release, capture denial, external release, empty
settings fallback, and generation teardown. Mock scene tests alone do not
prove an interactive palette or settings-driven entries on the strict host.

## Alternatives considered

A bundled-only palette was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved the palette to an independent first-party package.

## Affected contracts

No `entries` schema is accepted here. Generic plain-table extraction by the
configuration VM is not a manifest settings-kind extension.

## Open points

[Issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90) owns
declarations and table-shaped entry reconciliation. Live query typing and
provider enumeration need separately owned integration work.

## Acceptance criteria

Document only the pinned session lifecycle and empty-default behavior until
settings and interactive input have independent implementation evidence.

## P0 Review Sign-off

No new security acceptance is claimed by this source synchronization.

## References

- [Palette plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
