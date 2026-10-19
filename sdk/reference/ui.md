---
title: bitty.ui Reference
description: L2 declarative UI contribution surface with painted edge bands a focusable overlay host and typed denials
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 130
---

# bitty.ui Reference

> Status: **draft**. Normative surface detail lives in the accepted
> [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md);
> executable behavior lives in the `bitty` repository: slot validation in
> `crates/bitty-lua/src/ui.rs`, the bridge and default host in
> `crates/bitty-lua/src/host.rs`, and the host placement policy, mount gate,
> and band routing in `crates/bitty-runtime/src/runtime/band_slots.rs` plus
> `crates/bitty-runtime/src/plugin_runtime/services.rs`, all at
> `bitty@da63e42e` (bitty PR #2234, CTX-1341; overlay host CTX-0941;
> `statusline` removal CTX-1140, bitty PR #1962). This page states no behavior
> beyond what those sources pin.

## Purpose and scope

`bitty.ui` is the accepted L2 surface for declarative slot contributions:
plugins mount scene subtrees into host-owned slots and update them by handle,
altering presentation but never terminal truth (RFC "UI contributions (L2)").
The `bitty` runtime host implements `ui.mount` and `ui.update` through a single
placement policy, `ui_slot_placement`
(`crates/bitty-runtime/src/runtime/band_slots.rs:87`, `bitty@da63e42e`). The
mount gate and the chrome-band routing both call that policy, so a slot is
never admitted at mount time and then dropped or ignored at render time
(`band_slots.rs` module docs).

| Slot         | Current Core host at `bitty@da63e42e`                                                                                                                                                           |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `top`        | Top edge band, painted.                                                                                                                                                                         |
| `bottom`     | Bottom edge band, painted. Status presentation composes here (bar replacement).                                                                                                                 |
| `overlay`    | Core focusable-overlay host (CTX-0941): the block is retained, is not a band, and may own the transient input capture through `bitty.ui.overlay.*`.                                             |
| `left`       | Accepted name only. No vertical band host: `ui.mount` fails closed with typed `E_UI_UNAVAILABLE`; no handle, no block, and no reserved columns.                                                 |
| `right`      | Accepted name only. No vertical band host: `ui.mount` fails closed with typed `E_UI_UNAVAILABLE`; no handle, no block, and no reserved columns.                                                 |
| `tabline`    | Accepted name, unavailable: reserved for future panel tabs (PW-10); fails with `E_UI_UNAVAILABLE` after the exclusive-claim gate.                                                               |
| `terminal`   | Accepted name, unavailable: no terminal-attached block host; fails with `E_UI_UNAVAILABLE`.                                                                                                     |
| `statusline` | **Not a slot**. Removed by Core in CTX-1140 (bitty PR #1962); `ui.mount` rejects the name with `E_UI_COMPONENT_INVALID` before the host call. Status presentation composes on `bottom` instead. |

Painted bands stack from the window edge inward in plugin id byte order; a
mount order change does not move a surface between plugins, while one
plugin's own mounts on the same edge keep their mount order. Core draws no
workspace display of its own, so there is no Core band reservation; visible
plugin bands shrink the layout container through `band_exclusive_container`
(CTX-0946 C3), so no terminal cell is painted under a band. Known gaps remain:
several mounts from one plugin on one edge are all kept instead of enforcing
the candidate one-surface-per-edge rule, and `chrome.<edge>.order` is not wired
(`band_slots.rs` module docs).

### Accepted RFC divergence

The accepted [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
still lists `statusline` in its frozen closed slot set and describes status
components as subtrees mounted in that slot (RFC "UI contributions (L2)"). The
accepted [Plugin Platform RFC](../../specifications/plugin-platform-rfc.md)
restates the same frozen vocabulary. This page does not rewrite either accepted
source.

Current Core behavior diverges: CTX-1140 removed `statusline` as a breaking
change, so the name fails as an unknown slot and status presentation is a
`bottom`-band composition. Until the accepted text is revised, treat the RFC
vocabulary as the accepted contract record and this page plus the cited Core
sources as the current host fact. `left` and `right` remain accepted names in
both the RFC and Core; only their host behavior changed (they now fail closed
instead of storing an unpainted block).

## Signature

The accepted RFC vocabulary names `bitty.ui.mount(slot, component)` and
`bitty.ui.update(block_id, component)` (RFC "UI contributions (L2)"). Both are
callable against the `bitty` runtime host: `PluginServices::ui_mount` and
`PluginServices::ui_update`
(`crates/bitty-runtime/src/plugin_runtime/services.rs:2800` and `:2848`,
`bitty@da63e42e`).

A mount into `top`, `bottom`, or `overlay` returns a `block_id`. A mount into
`left`, `right`, `tabline`, or `terminal` fails closed with
`E_UI_UNAVAILABLE` after the capability and claim gates, with a message naming
the slot and the host-specific reason (`ui_slot_placement`, `band_slots.rs:87`;
`unsupported_slot_error`, `band_slots.rs:112`). A mount on `statusline` fails
earlier, in the bridge slot check, with `E_UI_COMPONENT_INVALID`.

A bare host that supplies no UI surface still uses the `HostServices` trait
defaults, which fail every call closed with `E_UI_UNAVAILABLE`, class `runtime`
(`crates/bitty-lua/src/host.rs`, `bitty@da63e42e`): `ui_mount` at `host.rs:1085`
(`"host has no ui.mount surface"`) and `ui_update` at `host.rs:1117`
(`"host has no ui.update surface"`). Such a host can never gain ambient
authority from the always-present `bitty.ui` namespace (`host.rs` `ui_mount`
docs, LUA-OQ-2).

## Params

The current Core slot set is the accepted seven names
(`terminal | top | bottom | left | right | tabline | overlay`). The accepted
RFC still adds `statusline` to its frozen vocabulary; that eighth name is
rejected by this build (see
[Accepted RFC divergence](#accepted-rfc-divergence)). Acceptance into the set
does not imply a host surface: see the placement table in
[Purpose and scope](#purpose-and-scope).

The component is a declarative node table restricted for v1 to the `Text`,
`Row`, `Column`, and `List` nodes (RFC "UI contributions (L2)"). The bridge
validates the slot name and the component shape against the accepted v1
contract before the host call (`crates/bitty-lua/src/ui.rs` `UiSlot::parse`
and `read_component`; `crates/bitty-lua/src/host.rs` `ui_mount` docs and
`mount` callback, `bitty@da63e42e`). The update handle is an opaque,
generation-owned integer (`block_id`) (RFC "UI contributions (L2)" and
`host.rs` `ui_mount` docs).

## Returns

A successful mount into `top`, `bottom`, or `overlay` returns the opaque,
generation-owned `block_id` handle (`crates/bitty-lua/src/host.rs` `ui_mount`
docs, `bitty@da63e42e`). An update replaces the block's scene subtree under the
same `block_id` with an incremented version and returns a boolean: `true` for a
live handle and `false` for a stale or foreign handle (RFC "UI contributions
(L2)"; `host.rs` `ui_update` docs; pinned by
`probe_plugin_mounts_bottom_block` in `crates/bitty-runtime/tests/plugin_ui.rs`).

A rejected mount returns no handle. In particular, `left` and `right` cannot
return a handle that the renderer would never paint, reserve, or hit-test
(`vertical_band_slots_fail_closed_at_mount`, `bitty@da63e42e`).

## Errors

Denials arrive as catchable Lua error tables (`class` / `code` / `message`,
per `BridgeError::to_error` in `crates/bitty-lua/src/host.rs`,
`bitty@da63e42e`); match on `code`.

A `ui.mount` call is checked in this order at `bitty@da63e42e`:

1. Bridge: slot name and component shape (`E_UI_COMPONENT_INVALID`), then the
   pre-commit expiry check (`E_TIMEOUT`) (`host.rs` `mount` callback and
   `ui_mount_with_expiry`). An unknown name, including `statusline`, fails
   here.
2. Host capability gate: `ui.rich`, then `ui.overlay` for the `overlay` slot
   (`E_CAPABILITY_DENIED`) (`services.rs:2800`).
3. Host claim gate: the `tabline` exclusive claim (`E_UI_CLAIM_REQUIRED`)
   (`services.rs` `ui_mount`).
4. Host availability: an accepted slot this build does not present —
   `left`, `right`, `tabline`, or `terminal` — fails with
   `E_UI_UNAVAILABLE` (`services.rs` `ui_mount`, `band_slots.rs:87`).
5. Host block registry bounds (`E_UI_BLOCK_BUDGET`) (`services.rs`
   `UiBlocks`).

A missing capability or an unclaimed `tabline` therefore reports
`E_CAPABILITY_DENIED` or `E_UI_CLAIM_REQUIRED`, never `E_UI_UNAVAILABLE`.
`statusline` never reaches the host gates because step 1 rejects it.

| `code`                   | `class`      | When                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `E_UI_COMPONENT_INVALID` | `validation` | An unknown slot name (including `statusline`) or a component that violates the scene contract, checked in the bridge before the host call (`host.rs` `mount` callback; `crates/bitty-lua/src/ui.rs` `ui_slot_enum_is_the_closed_v1_set`; RFC "UI contributions (L2)", LUA-OQ-7; pinned by `ui_mount_rejects_unknown_slot_at_host_boundary`, `lua_band_slot_mounts_land_in_the_expected_band`, and `oversized_component_is_rejected_before_any_mount` in `crates/bitty-runtime/tests/plugin_ui.rs`).                                                                                                                                                                                         |
| `E_TIMEOUT`              | `budget`     | Pre-commit expiry: mounting and update commit through the check-then-act expiry path, so an expired call fails closed without mutating (`host.rs` `ui_mount_with_expiry` / `ui_update_with_expiry` docs).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `E_CAPABILITY_DENIED`    | `runtime`    | The `ui.rich` grant is absent, or `ui.overlay` is absent for the `overlay` slot (`services.rs` `ui_mount`; pinned by `mount_denied_without_ui_rich_is_typed` and `overlay_slot_requires_ui_overlay_grant` in `crates/bitty-runtime/tests/plugin_ui.rs`, and at the seam by `capability_denial_propagates_typed` in `crates/bitty-lua/tests/ui_bridge.rs`).                                                                                                                                                                                                                                                                                                                                  |
| `E_UI_CLAIM_REQUIRED`    | `validation` | `tabline` is mounted without a matching exclusive claim in the manifest's `[lazy].claims` (`services.rs` `ui_mount`; pinned by `tabline_slot_requires_exclusive_claim` in `crates/bitty-runtime/tests/plugin_ui.rs` and the `services.rs` unit test of the same name).                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `E_UI_UNAVAILABLE`       | `runtime`    | (a) An accepted slot this build does not present — `left`, `right`, `tabline`, or `terminal` — after the capability and claim gates pass; the message names the slot and the reason (`band_slots.rs:87`, `band_slots.rs:112`; pinned by `vertical_band_slots_fail_closed_at_mount` and `unsupported_error_is_typed` in `crates/bitty-runtime/tests/plugin_ui.rs`). (b) A bare host with no mount path at all, through the `HostServices` defaults (`host.rs:1085`, `host.rs:1117`; pinned by `host_without_ui_surface_fails_closed` in `crates/bitty-lua/tests/ui_bridge.rs`). `overlay` is **not** in this group: a granted overlay mount is hosted (`overlay_slot_with_grant_is_hosted`). |
| `E_UI_BLOCK_BUDGET`      | `budget`     | A host block registry cap is exceeded; a host diagnostic pending an accepted stable plugin-visible code, not part of the stable v1 vocabulary (`services.rs` `UiBlocks` docs; pinned by `mount_loop_hits_block_budget_fail_closed` in `crates/bitty-runtime/tests/plugin_ui.rs`).                                                                                                                                                                                                                                                                                                                                                                                                           |

The SDK machine-readable surface lists the `ui.mount` errors as
`E_CAPABILITY_DENIED`, `E_UI_CLAIM_REQUIRED`, `E_UI_UNAVAILABLE`, and
`E_UI_COMPONENT_INVALID` (`surface/bitty-plugin-api-v1.json`,
`bitty-plugin-sdk@5592ff6`). No new code was added to the v1 error vocabulary
for the band host: `E_UI_UNAVAILABLE` already existed for the bare-host case
(`band_slots.rs` module docs). The `statusline` removal reuses the existing
`E_UI_COMPONENT_INVALID` unknown-slot path rather than adding a code.

## Capability

`CapabilityFamily::Ui` (`crates/bitty-plugin-host/src/capability.rs`, `Ui`
variant; closed identifiers `ui.rich`, `ui.overlay`, `ui.overlay.focus`,
`ui.protocol-register`). The `bitty` runtime host enforces `ui.rich` for every
mount and update, `ui.overlay` additionally for the `overlay` slot, and an
exclusive `[lazy].claims` entry for `tabline` (`crates/bitty-runtime/src/plugin_runtime/services.rs`
`ui_mount` / `ui_update`, `bitty@da63e42e`; RFC "UI contributions (L2)").

The `overlay` slot is a Core focusable-overlay host, not a presentation-only
band: a block mounted there is retained in the generation registry and may
claim the single Core-owned transient input capture through
`bitty.ui.overlay.acquire` / `poll` / `release`. Capture is gated separately by
`ui.overlay.focus` (accepted W-01 v2, CTX-0941) and only a handle this
generation mounted into the `overlay` slot can own it. Safe mode and a host
without a capture surface fail those calls closed with `E_UI_UNAVAILABLE`.

Passing these gates does not make an unhosted slot mountable: `left`, `right`,
`tabline`, and `terminal` still fail with `E_UI_UNAVAILABLE` (see
[Errors](#errors)). Grants, families, and lifecycle belong to the capability
contract and are never redefined here ([Lua Reference Overview](overview.md)).

## Example

Band-hosted mount and update with `ui.rich` granted, verified against
`crates/bitty-runtime/tests/plugin_ui.rs` (`probe_plugin_mounts_bottom_block`,
`lua_band_slot_mounts_land_in_the_expected_band`, `bitty@da63e42e`):

```lua
-- Manifest grants ui.rich.
local handle = bitty.ui.mount("bottom", {
  kind = "Row",
  children = { { kind = "Text", text = "cwd:/tmp" } },
})
-- handle is a positive block_id; the block lands on the bottom band.
bitty.ui.update(handle, { kind = "Text", text = "v2" }) -- true
bitty.ui.update(handle + 1000, { kind = "Text", text = "x" }) -- false (foreign handle)

bitty.ui.mount("top", { kind = "Text", text = "top" }) -- top band
bitty.ui.mount("bottom", { kind = "Text", text = "status" }) -- status presentation composes here
```

A granted `overlay` mount is hosted and can own transient capture, verified
against `overlay_slot_with_grant_is_hosted` and
`focusable_overlay_slot_is_hosted_and_not_a_band` in
`crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@da63e42e`):

```lua
-- Manifest grants ui.rich, ui.overlay, and ui.overlay.focus.
local overlay = bitty.ui.mount("overlay", { kind = "Text", text = "pick" })
-- overlay is a retained block_id; it is not routed to a chrome band.
bitty.ui.overlay.acquire(overlay) -- starts the Core-owned capture session
```

Unavailable and removed names fail closed, verified against
`vertical_band_slots_fail_closed_at_mount`,
`tabline_slot_requires_exclusive_claim`, and
`ui_mount_rejects_unknown_slot_at_host_boundary` in
`crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@da63e42e`):

```lua
-- Manifest grants ui.rich and declares [lazy].claims = ["workspaceline"].
local ok, err = pcall(bitty.ui.mount, "left", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"; err.class == "runtime"
ok, err = pcall(bitty.ui.mount, "right", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"
ok, err = pcall(bitty.ui.mount, "tabline", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"
ok, err = pcall(bitty.ui.mount, "statusline", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_COMPONENT_INVALID" (not a slot)
ok, err = pcall(bitty.ui.mount, "terminal", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"
-- Without the claim, the tabline mount reports E_UI_CLAIM_REQUIRED instead.
```

Corroborating evidence: the `band_slots.rs` unit tests
`every_v1_slot_has_exactly_one_placement` (`band_slots.rs:226`),
`from_mounts_counts_unsupported_slots_instead_of_dropping_silently`
(`band_slots.rs:287`), and `unsupported_error_is_typed`
(`band_slots.rs:338`) at `bitty@da63e42e`.

## Limits

- Hosted subset: `top` and `bottom` are painted edge bands, and `overlay` is a
  Core focusable-overlay host that is not a band. `left`, `right`, `tabline`,
  and `terminal` fail with `E_UI_UNAVAILABLE`; `statusline` is not a slot and
  fails with `E_UI_COMPONENT_INVALID` (`band_slots.rs` module docs,
  `bitty@da63e42e`). All of this is host-build behavior in a `draft` reference
  and may change as further hosts land.
- No vertical bands exist in this build: `left` and `right` mounts return no
  handle, store no block, and reserve no columns
  (`vertical_band_slots_fail_closed_at_mount`, CTX-1341).
- Status presentation is not a separate slot. A status plugin composes its
  subtree on `bottom` (bar replacement); the name `statusline` is rejected.
- Visible plugin bands shrink the layout container, so no terminal cell is
  painted under a band. Several mounts from one plugin on one edge are still
  all kept; `chrome.<edge>.order` is not wired (`band_slots.rs` module docs).
- There are no global coordinates, shaders, pipelines, glyph injection,
  native windows, or renderer handles (RFC "UI contributions (L2)").
- `Image`, `CodeBlock`, `Table`, `Rule`, and bordered `Block` nodes are
  excluded from v1 (RFC "UI contributions (L2)").
- Status components are ordinary subtrees mounted on `bottom`, and popups are
  `overlay`-slot subtrees, not a new node kind (RFC "UI contributions (L2)").
- Out of scope (no source): band focus behavior, panel routing, and any
  composer behavior beyond the internal scene-diff step. Overlay focus and
  capture belong to the W-01 focusable-overlay host contract.

## Compatibility and versioning

Draft reference: this page states current host behavior and carries no
compatibility promise while its status is draft. The target contract is the
[Compatibility policy](../plugin-api-v1-lua-surface-rfc.md#compatibility-policy)
in the accepted Plugin API v1 Lua Surface RFC: the surface is stable within
`1.x`, additions ship as minor versions, and removals or narrowings require a
major version, gated by the manifest `compat.plugin-api` range against the
runtime `bitty.api_version`.

The `statusline` removal was a breaking change recorded by Core in CTX-1140
(bitty PR #1962). Plugins that compose status presentation on `bottom` do not
depend on the removed name. The accepted RFC vocabulary has not yet been
revised; the divergence is recorded in
[Accepted RFC divergence](#accepted-rfc-divergence).

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (accepted; "UI contributions (L2)" spellings, frozen slot set, v1 node set,
  `block_id` versioning, `ui.rich` / `ui.overlay` gates, LUA-OQ-7,
  LUA-OQ-11). Its `statusline` vocabulary and non-focusable overlay wording
  predate CTX-1140 and CTX-0941; see
  [Accepted RFC divergence](#accepted-rfc-divergence).
- [Plugin Platform RFC](../../specifications/plugin-platform-rfc.md)
  (accepted; restates the frozen slot vocabulary, including `statusline`).
- [Lua Reference Overview](overview.md) (error contract, capability gating,
  executability rule, L1/L2 split)
- [Lua API Reference](README.md) (route table)
- `bitty` `crates/bitty-runtime/src/runtime/band_slots.rs`
  (`bitty@da63e42e`, bitty PR #2234, CTX-1341; module docs placement table,
  stacking, exclusive zone, and known gaps; `ui_slot_placement` at
  `band_slots.rs:87`; `unsupported_slot_error` at `band_slots.rs:112`; tests
  `every_v1_slot_has_exactly_one_placement`,
  `from_mounts_counts_unsupported_slots_instead_of_dropping_silently`,
  `unsupported_error_is_typed`)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (`bitty@da63e42e`; `PluginServices::ui_mount` gate order at
  `services.rs:2800`; `ui_update` at `services.rs:2848`; overlay capture gates
  and `UiBlocks` registry bounds)
- `bitty` `crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@da63e42e`;
  `probe_plugin_mounts_bottom_block`,
  `lua_band_slot_mounts_land_in_the_expected_band`,
  `vertical_band_slots_fail_closed_at_mount`,
  `overlay_slot_with_grant_is_hosted`,
  `focusable_overlay_slot_is_hosted_and_not_a_band`,
  `mount_denied_without_ui_rich_is_typed`,
  `overlay_slot_requires_ui_overlay_grant`,
  `tabline_slot_requires_exclusive_claim`,
  `mount_loop_hits_block_budget_fail_closed`,
  `oversized_component_is_rejected_before_any_mount`)
- `bitty` `crates/bitty-lua/src/ui.rs` (`bitty@da63e42e`; `UiSlot` current
  seven-name set, `read_component`, `ui_slot_enum_is_the_closed_v1_set` pins
  the rejection of `statusline`)
- `bitty` `crates/bitty-lua/src/host.rs` (`bitty@da63e42e`; `ui_mount`
  default at `host.rs:1085`; `ui_update` default at `host.rs:1117`;
  `ui_mount_with_expiry` / `ui_update_with_expiry` pre-commit docs;
  `E_UI_UNAVAILABLE` constant; `bitty.ui.overlay.*` capture contract)
- `bitty` `crates/bitty-plugin-host/src/capability.rs`
  (`CapabilityFamily::Ui`; `ui.rich`, `ui.overlay`, `ui.overlay.focus`,
  `ui.protocol-register` closed identifiers)
- `bitty` `crates/bitty-lua/tests/ui_bridge.rs`
  (`host_without_ui_surface_fails_closed`,
  `probe_bitty_ui_namespace_is_present`,
  `mount_and_update_round_trip_validated_scene`,
  `capability_denial_propagates_typed`)
