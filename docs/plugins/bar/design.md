---
title: Bar plugin design
description: Current text-row bar policy and deferred unified-chrome design
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 51
---

# Bar plugin design

## Document status

Implementation snapshot at `f007a03dfa6036ff67458095dc71169c0c6ee7e3`, pinned
by registry #92. Current text-row behavior is distinct from the broader
candidate unified-chrome design; no release verification is claimed.

## Purpose and scope

Present workspace state and optional system context in one edge row while
Core owns workspace identity, lifecycle, geometry, and terminal truth.

## Normative sources this specification must not weaken

- [ADR 0014](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0014-workspace-core-presentation-plugins.md)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
- [Settings reference](../../../sdk/reference/settings.md)

## Terminology

A module produces bounded text atoms. A module group orders atoms; it does
not imply a separate alignment container. A workspace label is decorated
text, not a new scene-node kind.

## Current presentation and interaction

The plugin mounts `top` or `bottom` through `bitty.ui.mount` under `ui.rich`.
It composes a `Row` of `Text`, orders left/center/right groups sequentially,
and hides a lone workspace unless configured otherwise or a scratchpad is
occupied. Inactive workspace labels request focus through a registered
command and `workspace.control`. Detail text may use a grant-gated,
rate-limited notification fallback. Optional sensor, device, and audio
providers hide when absent or ungranted; read and control are separate.

The [schema page](schemas.md) enumerates real settings and triggers. Chassis
scalars and group string arrays are effective. Per-module maps remain inert
because their declarations are empty string arrays, not accepted table kinds.

True center/right layout, richer widgets, and broader workspace add/close or
wheel interactions remain candidate. Core's existing `Text`, `Row`, `Column`,
and `List` vocabulary is not extended by calling text labels capsules.

## Security review

UI handles belong to one generation. Settings supply values only; the host
checks capability, slot, provider, and resource limits. The bar never alters
terminal truth or executes on parser/render/input hot paths.

## Verification plan

Validate the pinned manifest and source, settings strict-type fallback,
workspace focus grants, optional-provider denial, and UI teardown. A live
presentation campaign is required before claiming end-to-end usability.

## Alternatives considered

Bundling presentation in Core conflicts with ADR 0014. Inventing a new widget
or map-valued settings kind would require a separately accepted contract;
current code keeps text composition and module defaults.

## Affected contracts

The manifest/settings and generation-owned declarative UI contracts remain
unchanged. The [unified-chrome candidate](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/sparse-workspace-unified-bar-candidate.md)
is direction, not an implementation inventory for this pin.

## Open points

[Official schema issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90)
still owns the per-module option shape. Alignment and richer presentation
need their own accepted surface and verification.

## Acceptance criteria

Claims of current behavior must match the pinned source and host integration.
Registry compatibility alone does not prove product or security verification.

## P0 Review Sign-off

No new security or contract acceptance is claimed by this synchronization.

## References

- [Schemas and contracts](schemas.md)
- [Evidence register](evidence.md)
- [Bar documentation](README.md)
