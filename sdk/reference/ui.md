---
title: bitty.ui Reference
description: L2 declarative UI contribution surface with band-hosted slots slot gates and typed denials
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
> executable behavior lives in the `bitty` repository: the bridge and
> default host in `crates/bitty-lua/src/host.rs` and the band host in
> `crates/bitty-runtime/src/runtime/band_slots.rs` plus
> `crates/bitty-runtime/src/plugin_runtime/services.rs`, all at
> `bitty@5670d9ae` (bitty PR #1609, CTX-0923). This page states no behavior
> beyond what those sources pin.

## Purpose and scope

`bitty.ui` is the accepted L2 surface for declarative slot contributions:
plugins mount scene subtrees into host-owned slots and update them by
handle, altering presentation but never terminal truth (RFC "UI
contributions (L2)"). The `bitty` runtime host implements `ui.mount` and
`ui.update` for the band-hosted slots through a single placement policy,
`ui_slot_placement` (`crates/bitty-runtime/src/runtime/band_slots.rs:86`,
`bitty@5670d9ae`, bitty PR #1609, CTX-0923). The mount gate and the
chrome-band routing both call that policy, so a slot is never admitted at
mount time and then dropped at render time (`band_slots.rs` module docs).

| Slot         | Host placement at `bitty@5670d9ae`                                                                    |
| ------------ | ----------------------------------------------------------------------------------------------------- |
| `top`        | Top edge band, painted.                                                                               |
| `bottom`     | Bottom edge band, painted.                                                                            |
| `statusline` | Bottom edge band, painted; shares one ordering with `bottom`.                                         |
| `left`       | Accepted and stored on the left edge; not painted until vertical bands ship.                          |
| `right`      | Accepted and stored on the right edge; not painted until vertical bands ship.                         |
| `tabline`    | Accepted v1 slot, unavailable: reserved for future panel tabs (PW-10); fails with `E_UI_UNAVAILABLE`. |
| `overlay`    | Accepted v1 slot, unavailable: no plugin overlay host yet; fails with `E_UI_UNAVAILABLE`.             |
| `terminal`   | Accepted v1 slot, unavailable: no terminal-attached block host yet; fails with `E_UI_UNAVAILABLE`.    |

Painted bands stack from the window edge inward in plugin id byte order,
independent of mount order, and start inward of the rows the Core
workspaceline band reserves on the same edge (`band_slots.rs` module docs).
The `band_slots.rs` module docs record the known gaps: plugin bands reserve
no exclusive zone and overlay the terminal content row they sit on,
several mounts from one plugin on one edge are all kept, and
`chrome.<edge>.order` is not wired. The status remains `draft`: the
host behavior above is implemented in this `bitty` build, not a frozen
compatibility guarantee.

## Signature

The accepted RFC vocabulary names `bitty.ui.mount(slot, component)` and
`bitty.ui.update(block_id, component)` (RFC "UI contributions (L2)"). Both
are callable against the `bitty` runtime host: `PluginServices::ui_mount`
and `PluginServices::ui_update`
(`crates/bitty-runtime/src/plugin_runtime/services.rs:1142`,
`bitty@5670d9ae`). A mount into a band-hosted slot (`top`, `bottom`,
`statusline`, `left`, `right`) returns a `block_id`; a mount into
`tabline`, `overlay`, or `terminal` fails closed with `E_UI_UNAVAILABLE`
after the capability and claim gates, with a message naming the slot and
the host-specific reason (`ui_slot_placement`, `band_slots.rs:86`;
`unsupported_slot_error`, `band_slots.rs:106`).

A bare host that supplies no UI surface still uses the `HostServices`
trait defaults, which fail every call closed with `E_UI_UNAVAILABLE`,
class `runtime` (`crates/bitty-lua/src/host.rs`, `bitty@5670d9ae`):
`ui_mount` at `host.rs:727` (`"host has no ui.mount surface"`) and
`ui_update` at `host.rs:759` (`"host has no ui.update surface"`). Such a
host can never gain ambient authority from the always-present `bitty.ui`
namespace (`host.rs` `ui_mount` docs, LUA-OQ-2).

## Params

The slot is an accepted closed set
(`terminal | top | bottom | left | right | tabline | statusline | overlay`)
(RFC "UI contributions (L2)"). Acceptance into the set does not imply a
host surface: see the placement table in
[Purpose and scope](#purpose-and-scope). The component is a declarative
node table restricted for v1 to the `Text`, `Row`, `Column`, and `List`
nodes (RFC "UI contributions (L2)"). The bridge validates the slot name
and the component shape against the accepted v1 contract before the host
call (`crates/bitty-lua/src/host.rs` `ui_mount` docs and `mount` callback,
`bitty@5670d9ae`). The update handle is an opaque, generation-owned
integer (`block_id`) (RFC "UI contributions (L2)" and `host.rs`
`ui_mount` docs).

## Returns

A successful mount returns the opaque, generation-owned `block_id` handle
(`crates/bitty-lua/src/host.rs` `ui_mount` docs, `bitty@5670d9ae`). An
update replaces the block's scene subtree under the same `block_id` with
an incremented version and returns a boolean: `true` for a live handle and
`false` for a stale or foreign handle (RFC "UI contributions (L2)";
`host.rs` `ui_update` docs; pinned by `probe_plugin_mounts_statusline_block`
in `crates/bitty-runtime/tests/plugin_ui.rs`).

## Errors

Denials arrive as catchable Lua error tables (`class` / `code` / `message`,
per `BridgeError::to_error` in `crates/bitty-lua/src/host.rs`,
`bitty@5670d9ae`); match on `code`.

A `ui.mount` call is checked in this order at `bitty@5670d9ae`:

1. Bridge: slot name and component shape (`E_UI_COMPONENT_INVALID`), then
   the pre-commit expiry check (`E_TIMEOUT`) (`host.rs` `mount` callback
   and `ui_mount_with_expiry`).
2. Host capability gate: `ui.rich`, then `ui.overlay` for the `overlay`
   slot (`E_CAPABILITY_DENIED`) (`services.rs:1142`).
3. Host claim gate: the `tabline` exclusive claim (`E_UI_CLAIM_REQUIRED`)
   (`services.rs` `ui_mount`).
4. Host availability: an accepted slot this build does not present
   (`E_UI_UNAVAILABLE`) (`services.rs` `ui_mount`, `band_slots.rs:86`).
5. Host block registry bounds (`E_UI_BLOCK_BUDGET`) (`services.rs`
   `UiBlocks`).

A missing capability or an unclaimed `tabline` therefore reports
`E_CAPABILITY_DENIED` or `E_UI_CLAIM_REQUIRED`, never `E_UI_UNAVAILABLE`.

| `code`                   | `class`      | When                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `E_UI_COMPONENT_INVALID` | `validation` | An unknown slot name or a component that violates the scene contract, checked in the bridge before the host call (`host.rs` `mount` callback; RFC "UI contributions (L2)", LUA-OQ-7; pinned by `lua_band_slot_mounts_land_in_the_expected_band` and `oversized_component_is_rejected_before_any_mount` in `crates/bitty-runtime/tests/plugin_ui.rs`).                                                                                                                                                                                                                                                        |
| `E_TIMEOUT`              | `budget`     | Pre-commit expiry: mounting and update commit through the check-then-act expiry path, so an expired call fails closed without mutating (`host.rs` `ui_mount_with_expiry` / `ui_update_with_expiry` docs).                                                                                                                                                                                                                                                                                                                                                                                                    |
| `E_CAPABILITY_DENIED`    | `runtime`    | The `ui.rich` grant is absent, or `ui.overlay` is absent for the `overlay` slot (`services.rs` `ui_mount`; pinned by `mount_denied_without_ui_rich_is_typed` and `overlay_slot_requires_ui_overlay_grant` in `crates/bitty-runtime/tests/plugin_ui.rs`, and at the seam by `capability_denial_propagates_typed` in `crates/bitty-lua/tests/ui_bridge.rs`).                                                                                                                                                                                                                                                   |
| `E_UI_CLAIM_REQUIRED`    | `validation` | `tabline` is mounted without a matching exclusive claim in the manifest's `[lazy].claims` (`services.rs` `ui_mount`; pinned by `tabline_slot_requires_exclusive_claim` in `crates/bitty-runtime/tests/plugin_ui.rs` and the `services.rs` unit test of the same name).                                                                                                                                                                                                                                                                                                                                       |
| `E_UI_UNAVAILABLE`       | `runtime`    | (a) Primary live case: an accepted v1 slot this host build does not present — `tabline`, `overlay`, or `terminal` — after the capability and claim gates pass; the message names the slot and the reason (`band_slots.rs:86`, `band_slots.rs:106`; pinned by `unsupported_error_is_typed` and by `overlay_slot_with_grant_fails_closed_as_unhosted` in `crates/bitty-runtime/tests/plugin_ui.rs`). (b) A bare host with no mount path at all, through the `HostServices` defaults (`host.rs:727`, `host.rs:759`; pinned by `host_without_ui_surface_fails_closed` in `crates/bitty-lua/tests/ui_bridge.rs`). |
| `E_UI_BLOCK_BUDGET`      | `budget`     | A host block registry cap is exceeded; a host diagnostic pending an accepted stable plugin-visible code, not part of the stable v1 vocabulary (`services.rs` `UiBlocks` docs; pinned by `mount_loop_hits_block_budget_fail_closed` in `crates/bitty-runtime/tests/plugin_ui.rs`).                                                                                                                                                                                                                                                                                                                            |

The SDK machine-readable surface lists the `ui.mount` errors as
`E_CAPABILITY_DENIED`, `E_UI_CLAIM_REQUIRED`, `E_UI_UNAVAILABLE`, and
`E_UI_COMPONENT_INVALID` (`surface/bitty-plugin-api-v1.json`,
`bitty-plugin-sdk@5592ff6`). No new code was added to the v1 error
vocabulary for the band host: `E_UI_UNAVAILABLE` already existed for the
bare-host case (`band_slots.rs` module docs).

## Capability

`CapabilityFamily::Ui` (`crates/bitty-plugin-host/src/capability.rs`,
`Ui` variant; closed identifiers `ui.rich`, `ui.overlay`,
`ui.protocol-register`). The `bitty` runtime host enforces `ui.rich` for
every mount and update, `ui.overlay` additionally for the `overlay` slot,
and an exclusive `[lazy].claims` entry for `tabline`
(`crates/bitty-runtime/src/plugin_runtime/services.rs` `ui_mount` /
`ui_update`, `bitty@5670d9ae`; RFC "UI contributions (L2)"). Passing these
gates does not make `tabline` or `overlay` mountable in this build: both
still fail with `E_UI_UNAVAILABLE` (see [Errors](#errors)). The `overlay`
slot is presentation-only, non-focusable declarative content (RFC "UI
contributions (L2)", LUA-OQ-11). Grants, families, and lifecycle belong to
the capability contract and are never redefined here
([Lua Reference Overview](overview.md)).

## Example

Band-hosted mount and update with `ui.rich` granted, verified against
`crates/bitty-runtime/tests/plugin_ui.rs` (`probe_plugin_mounts_statusline_block`,
`lua_band_slot_mounts_land_in_the_expected_band`, `bitty@5670d9ae`):

```lua
-- Manifest grants ui.rich.
local handle = bitty.ui.mount("statusline", {
  kind = "Row",
  children = { { kind = "Text", text = "cwd:/tmp" } },
})
-- handle is a positive block_id; the block lands on the bottom band.
bitty.ui.update(handle, { kind = "Text", text = "v2" }) -- true
bitty.ui.update(handle + 1000, { kind = "Text", text = "x" }) -- false (foreign handle)

bitty.ui.mount("top", { kind = "Text", text = "top" }) -- top band
bitty.ui.mount("left", { kind = "Text", text = "left" }) -- stored, not painted yet
```

Unavailable slots fail closed even when claimed or granted, verified
against `tabline_slot_requires_exclusive_claim` and
`overlay_slot_with_grant_fails_closed_as_unhosted` in
`crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@5670d9ae`):

```lua
-- Manifest grants ui.rich and ui.overlay and declares [lazy].claims = ["tabline"].
local ok, err = pcall(bitty.ui.mount, "tabline", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"; err.class == "runtime"
ok, err = pcall(bitty.ui.mount, "overlay", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"
ok, err = pcall(bitty.ui.mount, "terminal", { kind = "Text", text = "x" })
-- ok == false; err.code == "E_UI_UNAVAILABLE"
-- Without the claim, the tabline mount reports E_UI_CLAIM_REQUIRED instead.
```

Corroborating evidence: the `band_slots.rs` unit tests
`every_v1_slot_has_exactly_one_placement` (`band_slots.rs:235`),
`from_mounts_counts_unsupported_slots_instead_of_dropping_silently`
(`band_slots.rs:281`), and `unsupported_error_is_typed`
(`band_slots.rs:326`) at `bitty@5670d9ae`; and the SDK mock host tests
`hosted and unavailable slots partition the accepted slot set`
(`tests/mock-host.test.ts:400`), `overlay stays unavailable even with
ui.overlay granted` (`tests/mock-host.test.ts:1926`), and `exclusive-claim
UI slots require a matching claim declaration`
(`tests/mock-host.test.ts:2234`) at `bitty-plugin-sdk@5592ff6`
(bitty-plugin-sdk PR #136, CTX-0064).

## Limits

- Hosted subset: only `top`, `bottom`, and `statusline` are painted;
  `left` and `right` are stored but not painted until vertical bands
  ship; `tabline`, `overlay`, and `terminal` fail with `E_UI_UNAVAILABLE`
  (`band_slots.rs` module docs, `bitty@5670d9ae`). All of this is
  host-build behavior in a `draft` reference and may change as further
  hosts land.
- Plugin bands reserve no exclusive zone yet and cover the terminal
  content row they sit on (`band_slots.rs` module docs, known gaps).
- There are no global coordinates, shaders, pipelines, glyph injection,
  native windows, or renderer handles (RFC "UI contributions (L2)").
- `Image`, `CodeBlock`, `Table`, `Rule`, and bordered `Block` nodes are
  excluded from v1 (RFC "UI contributions (L2)").
- Status components are ordinary subtrees mounted in the `statusline`
  slot, and popups are overlay-slot subtrees, not a new node kind (RFC "UI
  contributions (L2)").
- Out of scope (no source): focus behavior, panel routing, and any
  composer behavior beyond the internal scene-diff step.

## Compatibility and versioning

Draft reference: this page states the candidate v1 spellings and carries no
compatibility promise while its status is draft. The target contract is the
[Compatibility policy](../plugin-api-v1-lua-surface-rfc.md#compatibility-policy)
in the accepted Plugin API v1 Lua Surface RFC: the surface is stable within
`1.x`, additions ship as minor versions, and removals or narrowings require a
major version, gated by the manifest `compat.plugin-api` range against the
runtime `bitty.api_version`.

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (Accepted; "UI contributions (L2)" spellings, slot set, v1 node set,
  `block_id` versioning, `ui.rich` / `ui.overlay` gates, LUA-OQ-7,
  LUA-OQ-11)
- [Lua Reference Overview](overview.md) (error contract, capability gating,
  executability rule, L1/L2 split)
- [Lua API Reference](README.md) (route table; its `ui.md` row is pinned to
  `bitty@7da6d6f` evidence and predates the band host)
- `bitty` `crates/bitty-runtime/src/runtime/band_slots.rs`
  (`bitty@5670d9ae`, bitty PR #1609, CTX-0923; module docs placement table,
  stacking, Core reservation, and known gaps; `ui_slot_placement` at
  `band_slots.rs:86`; `unsupported_slot_error` at `band_slots.rs:106`;
  tests `every_v1_slot_has_exactly_one_placement`,
  `from_mounts_counts_unsupported_slots_instead_of_dropping_silently`,
  `unsupported_error_is_typed`)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (`bitty@5670d9ae`; `PluginServices::ui_mount` gate order at
  `services.rs:1142`; `UiBlocks` registry bounds and `E_UI_BLOCK_BUDGET`)
- `bitty` `crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@5670d9ae`;
  `probe_plugin_mounts_statusline_block`,
  `lua_band_slot_mounts_land_in_the_expected_band`,
  `mount_denied_without_ui_rich_is_typed`,
  `overlay_slot_requires_ui_overlay_grant`,
  `tabline_slot_requires_exclusive_claim`,
  `overlay_slot_with_grant_fails_closed_as_unhosted`,
  `mount_loop_hits_block_budget_fail_closed`,
  `oversized_component_is_rejected_before_any_mount`)
- `bitty` `crates/bitty-lua/src/host.rs` (`bitty@5670d9ae`; `ui_mount`
  default at `host.rs:727`; `ui_update` default at `host.rs:759`;
  `ui_mount_with_expiry` / `ui_update_with_expiry` pre-commit docs;
  `E_UI_UNAVAILABLE` constant)
- `bitty` `crates/bitty-plugin-host/src/capability.rs`
  (`CapabilityFamily::Ui`; `ui.rich`, `ui.overlay`,
  `ui.protocol-register` closed identifiers)
- `bitty` `crates/bitty-lua/tests/ui_bridge.rs`
  (`host_without_ui_surface_fails_closed`,
  `probe_bitty_ui_namespace_is_present`,
  `mount_and_update_round_trip_validated_scene`,
  `capability_denial_propagates_typed`)
- `bitty-plugin-sdk` mock host (`bitty-plugin-sdk@5592ff6`,
  bitty-plugin-sdk PR #136, CTX-0064; `tests/mock-host.test.ts` UI slot
  tests; `surface/bitty-plugin-api-v1.json` `ui.mount.errors`)
