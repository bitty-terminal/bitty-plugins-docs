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

Implementation snapshot at `0fa151395fe9d9d8b3b9dc4ba047eb13fa7b45c4`
(`0fa15139`), pinned by registry #115. Current text-row behavior is distinct
from the broader candidate unified-chrome design; no release verification is
claimed.

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
It composes a `Row` of `Text`, placing workspaces in the left group, the clock
centered across the full band, and provider-backed `network`, `cpu`, `memory`,
and `battery` modules at the right inset. Unavailable or ungranted provider
modules are omitted individually. A lone workspace is hidden unless configured
otherwise or a scratchpad is occupied, and a plan that resolves to nothing
falls back to the shipped default plan. Inactive workspace labels request focus
through a registered command and `workspace.control`; the `label` setting
selects index, name, or both. Workspace labels render as padded, filled capsules
rather than bracketed text; the exact marks, theme tokens, and square-corner
limit are specified in the [scene contract](schemas.md#scene-contract). Detail
text may use a grant-gated, rate-limited notification fallback. Optional
sensor, device, and audio providers hide when absent or ungranted; read and
control are separate. Settings are read at
activation and re-read on `config.reloaded`; only the `edge` selection remains
pinned to the generation.

The [schema page](schemas.md) enumerates real settings and triggers. Chassis
scalars and group string arrays are effective. Per-module maps remain inert
because their declarations are empty string arrays, not accepted table kinds.

The current source's left/center/right alignment is implemented, but richer
widgets and broader workspace add/close or wheel interactions remain candidate.
The broader unified-chrome candidate is a separate design direction, not an
implementation inventory for this pin. A workspace capsule remains a styled
`Text` atom; Core's existing `Text`, `Row`, `Column`, and `List` vocabulary is
not extended with a capsule node.

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
