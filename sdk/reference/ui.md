---
title: bitty.ui Reference
description: L2 declarative UI contribution surface with painted edge bands a focusable overlay host a presentation-only overlay and typed denials
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
> `crates/bitty-lua/src/host.rs`, the host placement policy, mount gate, and
> band routing in `crates/bitty-runtime/src/runtime/band_slots.rs` plus
> `crates/bitty-runtime/src/plugin_runtime/services.rs`, the band host in
> `crates/bitty-runtime/src/runtime/band_host.rs`, and the overlay paint path
> in `crates/bitty-runtime/src/runtime/present.rs`, all at
> `bitty@7d98f53a` (bitty PR #2234, CTX-1341; overlay host CTX-0941;
> `statusline` removal CTX-1140, bitty PR #1962; band/overlay style parity,
> typed validation, and band-only interaction routing CTX-1419, bitty PRs
> [#2385](https://github.com/bitty-terminal/bitty/pull/2385) `cd6b0724` and
> [#2392](https://github.com/bitty-terminal/bitty/pull/2392) `c816b00e`,
> issue #2191). This page states no behavior beyond what those sources pin.

## Purpose and scope

`bitty.ui` is the accepted L2 surface for declarative slot contributions:
plugins mount scene subtrees into host-owned slots and update them by handle,
altering presentation but never terminal truth (RFC "UI contributions (L2)").
The `bitty` runtime host implements `ui.mount` and `ui.update` through a single
placement policy, `ui_slot_placement`
(`crates/bitty-runtime/src/runtime/band_slots.rs:87`, `bitty@7d98f53a`). The
mount gate and the chrome-band routing both call that policy, so a slot is
never admitted at mount time and then dropped or ignored at render time
(`band_slots.rs` module docs).

| Slot         | Current Core host at `bitty@7d98f53a`                                                                                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `top`        | Top edge band, painted.                                                                                                                                                                                  |
| `bottom`     | Bottom edge band, painted. Status presentation composes here (bar replacement).                                                                                                                          |
| `overlay`    | Core overlay host (CTX-0941): the block is retained, is not a band, may be presented without capture through `bitty.ui.present.*`, and may own the transient input capture through `bitty.ui.overlay.*`. |
| `left`       | Accepted name only. No vertical band host: `ui.mount` fails closed with typed `E_UI_UNAVAILABLE`; no handle, no block, and no reserved columns.                                                          |
| `right`      | Accepted name only. No vertical band host: `ui.mount` fails closed with typed `E_UI_UNAVAILABLE`; no handle, no block, and no reserved columns.                                                          |
| `tabline`    | Accepted name, unavailable: reserved for future panel tabs (PW-10); fails with `E_UI_UNAVAILABLE` after the exclusive-claim gate.                                                                        |
| `terminal`   | Accepted name, unavailable: no terminal-attached block host; fails with `E_UI_UNAVAILABLE`.                                                                                                              |
| `statusline` | **Not a slot**. Removed by Core in CTX-1140 (bitty PR #1962); `ui.mount` rejects the name with `E_UI_COMPONENT_INVALID` before the host call. Status presentation composes on `bottom` instead.          |

Painted bands stack from the window edge inward in plugin id byte order; a
mount order change does not move a surface between plugins, while one
plugin's own mounts on the same edge keep their mount order. Core draws no
workspace display of its own, so there is no Core band reservation; visible
plugin bands shrink the layout container through `band_exclusive_container`
(CTX-0946 C3), so no terminal cell is painted under a band. Known gaps remain:
several mounts from one plugin on one edge are all kept instead of enforcing
the candidate one-surface-per-edge rule, and `chrome.<edge>.order` is not wired
(`band_slots.rs` module docs).

### Accepted contract amendment record

The accepted [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
was amended on 2026-10-26 through the owner amendment process (CarryCtx task
CTX-1546, [bitty issue #2676](https://github.com/bitty-terminal/bitty/issues/2676),
amendment reference `OQ-UI-CONTRACT-AMENDMENT`): the closed slot set is now the
seven names this page documents (`statusline` left the set), status composition
is a `bottom`-band fact, and the overlay wording separates the never-focusing
slot content from the separately accepted `W-01` capture surface
(`bitty.ui.overlay.*`, gate `ui.overlay.focus`). The removal evidence recorded
here predates the amendment: CTX-1140 removed `statusline` from the Core slot
vocabulary as a breaking change (bitty PR #1962), and the plugin SDK dropped the
retired name under CTX-1570. The [Plugin Platform
RFC](../../specifications/plugin-platform-rfc.md) was amended in the same change.
`left` and `right` remain accepted names with changed host behavior (they fail
closed instead of storing an unpainted block).

The same contract process owns the accepted wording for the parity record:
CTX-1419 (bitty PRs #2385 and #2392) completed the implementation evidence
reconciled in this page — band/overlay style parity, typed validation,
per-generation bounds, presentation-only overlay mounting with the separately
gated `ui.overlay.focus` capture, and band-only interaction routing. The bounded
composition API design track (CTX-1299) and the release-scope reconciliation
track (CTX-1305) stay open under their own tasks and are neither closed nor
amended by this page.

## Signature

The accepted RFC vocabulary names `bitty.ui.mount(slot, component)` and
`bitty.ui.update(block_id, component)` (RFC "UI contributions (L2)"). Both are
callable against the `bitty` runtime host: `PluginServices::ui_mount` and
`PluginServices::ui_update`
(`crates/bitty-runtime/src/plugin_runtime/services.rs:2800` and `:2848`,
`bitty@7d98f53a`).

A mount into `top`, `bottom`, or `overlay` returns a `block_id`. A mount into
`left`, `right`, `tabline`, or `terminal` fails closed with
`E_UI_UNAVAILABLE` after the capability and claim gates, with a message naming
the slot and the host-specific reason (`ui_slot_placement`, `band_slots.rs:87`;
`unsupported_slot_error`, `band_slots.rs:112`). A mount on `statusline` fails
earlier, in the bridge slot check, with `E_UI_COMPONENT_INVALID`.

A bare host that supplies no UI surface still uses the `HostServices` trait
defaults, which fail every call closed with `E_UI_UNAVAILABLE`, class `runtime`
(`crates/bitty-lua/src/host.rs`, `bitty@7d98f53a`): `ui_mount` at `host.rs:1085`
(`"host has no ui.mount surface"`) and `ui_update` at `host.rs:1117`
(`"host has no ui.update surface"`). Such a host can never gain ambient
authority from the always-present `bitty.ui` namespace (`host.rs` `ui_mount`
docs, LUA-OQ-2).

## Params

The current Core slot set is the accepted seven names
`terminal | top | bottom | left | right | tabline | overlay`. The 2026-10-26
contract amendment dropped the retired `statusline` name from the accepted
vocabulary; earlier factual pages recorded it as a divergence — see
[Accepted contract amendment record](#accepted-contract-amendment-record).
Acceptance into the set does not imply a host surface: see the placement table in
[Purpose and scope](#purpose-and-scope).

The component is a declarative node table restricted for v1 to the `Text`,
`Row`, `Column`, and `List` nodes (RFC "UI contributions (L2)"). The bridge
validates the slot name and the component shape against the accepted v1
contract before the host call (`crates/bitty-lua/src/ui.rs` `UiSlot::parse`
and `read_component`; `crates/bitty-lua/src/host.rs` `ui_mount` docs and
`mount` callback, `bitty@7d98f53a`). The update handle is an opaque,
generation-owned integer (`block_id`) (RFC "UI contributions (L2)" and
`host.rs` `ui_mount` docs).

## Component shape, style parity, and band interaction

### Nodes, styles, and typed validation

The bridge accepts exactly the four v1 kinds `Text`, `Row`, `Column`, and
`List` and rejects `Block`, `Image`, `CodeBlock`, `Table`, and `Rule`. Every
kind additionally accepts optional declarative style and interaction fields,
all validated in the bridge before any host call
(`crates/bitty-lua/src/ui.rs`, `bitty@7d98f53a`):

- `fg` / `bg`: bounded lowercase theme-token names. The host resolves the
  closed token table `foreground`, `background`, `cursor`, `selection`, and
  `accent`; another well-formed token name falls back to the span default and
  is counted in the host diagnostics. Fail-closed rejection of unknown tokens
  is deferred to the
  [Theme Token Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/theme-token-contract-candidate.md)
  (candidate); raw colour strings are not accepted.
- `bold`: a Boolean, painted with the synthetic double-strike the band host
  uses.
- `on_click` / `on_scroll`: declarative command bindings
  (`{ command, args }`) with an optional per-item
  `region = { id, kind = "item" | "module" }` identity. Command ids are at
  most 64 bytes, string argument values at most 256 bytes, and region ids at
  most 64 bytes. The dispatched argument table is capped at 16 entries, with
  host-injected args reserved out of that cap: an ordinary `on_click` binding
  may declare up to 16 arguments, an `on_scroll` binding reserves `delta` and
  declares at most 15, and a region-aware binding reserves `region` and
  `button` (click) or `delta` (scroll), declaring at most 14. Declaring
  different regions on the two bindings rejects the mount.

Validation is typed and fail-closed: an unknown or excluded kind, a malformed
text, children, style, or binding shape, and out-of-bounds values fail before
any host call with `E_UI_COMPONENT_INVALID`
(`unsupported_components_fail_typed` and `bounds_fail_closed` in
`crates/bitty-runtime/tests/lua_ui_parity.rs`).

### Band/overlay style parity

The two hosted surfaces, band (`top`/`bottom`) and overlay, paint the same
style vocabulary. Overlay rows flatten through the same band-host flattening
and resolve `fg`/`bg` through the same closed theme-token table as bands,
apply the same synthetic bold double-strike, and count unknown tokens through
the same host counter (bitty PR #2385, CTX-1419, issue #2191;
`paint_plugin_overlay` / `overlay_scene_rows` and the
`paint_plugin_overlay_tests` module in `crates/bitty-runtime/src/runtime/present.rs`,
`bitty@7d98f53a`):

| Aspect                   | Current host fact at `bitty@7d98f53a`                                                                                                                                                                                       |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Unstyled overlay row     | Keeps the pre-parity single-pill paint path byte-for-byte: one default fill and one text call (existing composer and palette scenes are unchanged).                                                                         |
| Styled overlay row       | Paints per flattened run with the resolved token colours and the bold double-strike; unknown token names fall back to the span default and add to the host diagnostic count.                                                |
| Empty styled overlay row | Still paints the one-cell default pill, so a live overlay surface never keeps keyboard capture while painting nothing (fail-open chrome, CTX-1143; bitty PR #2392; pinned by `styled_empty_overlay_row_keeps_pill_chrome`). |
| Overlay root declaration | A direct `Column`/`List` root's own `fg`/`bg`/`bold` declaration reaches its rows, matching band container inheritance (bitty PR #2392; pinned by `overlay_root_declaration_reaches_rows`).                                 |

### Band-only interaction routing and overlay capture

- Band-only routing: declared `on_click`/`on_scroll` bindings and per-item
  `region` identities dispatch only from band surfaces (`top`/`bottom`).
  Region rectangles derive from the painted span geometry, clip to the
  window, carry declared command ids only (no coordinates), and merge the
  host-owned `region`/`button`/`delta` entries last; a legacy span without a
  region keeps its primary-button-only payload, foreign qualifiers are
  denied, and no plugin VM runs on the input path
  (`click_and_scroll_route_through_commands`,
  `legacy_span_keeps_primary_button_and_args`,
  `regions_are_derived_from_painted_spans`, `hot_path_is_pure_data`, and
  `chrome_gestures_keep_terminal_authority` in
  `crates/bitty-runtime/tests/lua_ui_parity.rs`; chrome click/scroll never
  moves focus, never changes layout, and never starts a selection).
- Overlay surfaces: scene-node bindings are accepted and retained but not
  routed. There is no accepted overlay hit-test/geometry contract, so overlay
  pointer routing stays a design note and is not promoted here. Overlay
  interaction goes through the separately gated `bitty.ui.overlay.*` capture
  API (`ui.overlay.focus`), which is distinct from presentation: the mount
  itself presents declarative content only, and only an acquired capture
  session receives input (see [Capability](#capability)).

### Per-generation bounds

The accepted scene budgets are enforced per plugin generation as a strict
subset; terminal-wide aggregation across plugins is not implemented and stays
deferred (`crates/bitty-lua/src/ui.rs` budget table and
`crates/bitty-runtime/src/plugin_runtime/services.rs` block registry,
`bitty@7d98f53a`):

| Dimension             | Per-generation bound | Tie-in                                                                                                                                                |
| --------------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Component depth       | 16                   | Deliberately stricter bridge budget than the accepted `SCN-2` (32); the accepted scene-level bound still holds in presentation.                       |
| Nodes                 | 2048                 | Accepted `SCN-1`.                                                                                                                                     |
| Text bytes per block  | 256 KiB              | Accepted `SCN-3`.                                                                                                                                     |
| Retained blocks       | 64                   | Accepted `SCN-5` number, enforced per generation.                                                                                                     |
| Retained string bytes | 2 MiB aggregate      | Accepted `SCN-4` number, charged per generation for every host-retained string (text, theme tokens, command ids, argument keys and values, CTX-1340). |

Depth, node, and text bounds fail closed at the bridge with
`E_UI_COMPONENT_INVALID`; block and aggregate registry bounds fail closed at
the host with `E_UI_BLOCK_BUDGET` (see [Errors](#errors)).

## Returns

A successful mount into `top`, `bottom`, or `overlay` returns the opaque,
generation-owned `block_id` handle (`crates/bitty-lua/src/host.rs` `ui_mount`
docs, `bitty@7d98f53a`). An update replaces the block's scene subtree under the
same `block_id` with an incremented version and returns a boolean: `true` for a
live handle and `false` for a stale or foreign handle (RFC "UI contributions
(L2)"; `host.rs` `ui_update` docs; pinned by
`probe_plugin_mounts_bottom_block` in `crates/bitty-runtime/tests/plugin_ui.rs`).

A rejected mount returns no handle. In particular, `left` and `right` cannot
return a handle that the renderer would never paint, reserve, or hit-test
(`vertical_band_slots_fail_closed_at_mount`, `bitty@7d98f53a`).

## Errors

Denials arrive as catchable Lua error tables (`class` / `code` / `message`,
per `BridgeError::to_error` in `crates/bitty-lua/src/host.rs`,
`bitty@7d98f53a`); match on `code`.

A `ui.mount` call is checked in this order at `bitty@7d98f53a`:

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
| `E_UI_COMPONENT_INVALID` | `validation` | An unknown slot name (including `statusline`) or a component that violates the scene contract, checked in the bridge before the host call (`host.rs` `mount` callback; `crates/bitty-lua/src/ui.rs` `ui_slot_enum_is_the_closed_v1_set`; RFC "UI contributions (L2)", LUA-OQ-7; pinned by `lua_band_slot_mounts_land_in_the_expected_band` and `oversized_component_is_rejected_before_any_mount` in `crates/bitty-runtime/tests/plugin_ui.rs` and at the host seam by the `ui_mount_rejects_unknown_slot_at_host_boundary` unit test in `crates/bitty-runtime/src/plugin_runtime/services.rs`).                                                                                            |
| `E_TIMEOUT`              | `budget`     | Pre-commit expiry: mounting and update commit through the check-then-act expiry path, so an expired call fails closed without mutating (`host.rs` `ui_mount_with_expiry` / `ui_update_with_expiry` docs).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `E_CAPABILITY_DENIED`    | `runtime`    | The `ui.rich` grant is absent, or `ui.overlay` is absent for the `overlay` slot (`services.rs` `ui_mount`; pinned by `mount_denied_without_ui_rich_is_typed` and `overlay_slot_requires_ui_overlay_grant` in `crates/bitty-runtime/tests/plugin_ui.rs`, and at the seam by `capability_denial_propagates_typed` in `crates/bitty-lua/tests/ui_bridge.rs`).                                                                                                                                                                                                                                                                                                                                  |
| `E_UI_CLAIM_REQUIRED`    | `validation` | `tabline` is mounted without a matching exclusive claim in the manifest's `[lazy].claims` (`services.rs` `ui_mount`; pinned by `tabline_slot_requires_exclusive_claim` in `crates/bitty-runtime/tests/plugin_ui.rs` and the `services.rs` unit test of the same name).                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `E_UI_UNAVAILABLE`       | `runtime`    | (a) An accepted slot this build does not present — `left`, `right`, `tabline`, or `terminal` — after the capability and claim gates pass; the message names the slot and the reason (`band_slots.rs:87`, `band_slots.rs:112`; pinned by `vertical_band_slots_fail_closed_at_mount` and `unsupported_error_is_typed` in `crates/bitty-runtime/tests/plugin_ui.rs`). (b) A bare host with no mount path at all, through the `HostServices` defaults (`host.rs:1085`, `host.rs:1117`; pinned by `host_without_ui_surface_fails_closed` in `crates/bitty-lua/tests/ui_bridge.rs`). `overlay` is **not** in this group: a granted overlay mount is hosted (`overlay_slot_with_grant_is_hosted`). |
| `E_UI_BLOCK_BUDGET`      | `budget`     | A host block registry cap is exceeded (`services.rs` `UiBlocks`; pinned by `mount_loop_hits_block_budget_fail_closed` in `crates/bitty-runtime/tests/plugin_ui.rs`). The host emits it as the same catchable `class`/`code`/`message` table as every other denial at `bitty@75ec169f`, and the SDK declares it as a stable v1 code in `ui.mount.errors` and `ui.update.errors` (bitty-plugin-sdk#154). The `services.rs` comment still calls it pending an accepted plugin-visible code; the SDK surface is the plugin-facing commitment.                                                                                                                                                   |

The SDK machine-readable surface declares both `ui.mount` and `ui.update`
errors as `E_CAPABILITY_DENIED`, `E_UI_COMPONENT_INVALID`,
`E_UI_BLOCK_BUDGET`, and `E_TIMEOUT`, with `E_UI_CLAIM_REQUIRED` and
`E_UI_UNAVAILABLE` additionally on `ui.mount`
(`surface/bitty-plugin-api-v1.json`, bitty-plugin-sdk#154). The budget code is
therefore part of the stable plugin-facing v1 vocabulary, not a pending
diagnostic; `E_TIMEOUT` remains the generic bounded-call deadline code and is
already listed for other bounded surfaces. No code was renamed and no other v1
semantics changed. The `statusline` removal reuses the existing
`E_UI_COMPONENT_INVALID` unknown-slot path rather than adding a code.

## Capability

`CapabilityFamily::Ui` (`crates/bitty-plugin-host/src/capability.rs`, `Ui`
variant; closed identifiers `ui.rich`, `ui.overlay`, `ui.overlay.focus`,
`ui.present`, `ui.protocol-register`). The `bitty` runtime host enforces `ui.rich` for every
mount and update, `ui.overlay` additionally for the `overlay` slot, and an
exclusive `[lazy].claims` entry for `tabline` (`crates/bitty-runtime/src/plugin_runtime/services.rs`
`ui_mount` / `ui_update`, `bitty@7d98f53a`; RFC "UI contributions (L2)").

The `overlay` slot is a Core overlay host, not a presentation-only
band: a block mounted there is retained in the generation registry and may
claim the single Core-owned transient input capture through
`bitty.ui.overlay.acquire` / `poll` / `release`. Capture is gated separately by
`ui.overlay.focus` (accepted W-01 v2, CTX-0941) and only a handle this
generation mounted into the `overlay` slot can own it. Safe mode and a host
without a capture surface fail those calls closed with `E_UI_UNAVAILABLE`.

The same retained block can also be presented without capture: that path is
`ui.present` and is independent of `ui.overlay.focus` in both directions (see
[Presentation-only overlay](#presentation-only-overlay-uipresent)).

Passing these gates does not make an unhosted slot mountable: `left`, `right`,
`tabline`, and `terminal` still fail with `E_UI_UNAVAILABLE` (see
[Errors](#errors)). Grants, families, and lifecycle belong to the capability
contract and are never redefined here ([Lua Reference Overview](overview.md)).

The `ui.present` head is also listed in `CLOSED_CAPABILITY_HEADS`
(`crates/bitty-package/src/manifest.rs`, `bitty@8108e4f7`), so a manifest naming
it validates. Its effect statement is "Draw a non-focusable presentation panel
above the focused view (no input capture)", and it is deliberately separate from
the two `overlay` heads: no head implies another, so a manifest cannot gain
input authority by asking to draw (see
[Presentation-only overlay](#presentation-only-overlay-uipresent)).

## Presentation-only overlay (`ui.present`)

A block mounted into `overlay` can be drawn above the focused view with no input
authority at all: which-key hints, a read-only help panel, a transient status
card. `bitty.ui.present.show` and `bitty.ui.present.hide` are the named spelling
of that session at `bitty@8108e4f7`; `bitty.ui.overlay.show` and
`bitty.ui.overlay.hide` remain the CTX-1431 spelling of the same Core session and
are gated by the same head (`crates/bitty-lua/src/host.rs`, `present` and
`overlay` tables; `crates/bitty-runtime/src/plugin_runtime/services.rs`
`require_presentation_overlay`).

```lua
-- Manifest grants ui.rich, ui.overlay, and ui.present.
local handle = bitty.ui.mount("overlay", { kind = "Text", text = "which-key" })
bitty.ui.present.show(handle) -- true: drawn above the focused view, no capture
bitty.ui.present.hide(handle) -- true: the surface was mine and is now removed
bitty.ui.present.hide(handle) -- false: nothing of mine was live
```

### Admission and placement

| Property        | Behavior at `bitty@8108e4f7`                                                                                                                                                                                                                                                                                                            |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Grant           | `ui.present` alone authorizes drawing. `ui.rich` + `ui.overlay` mount the block and `ui.overlay.focus` captures input; none implies another and `ui.present` never implies capture (`services.rs` `require_presentation_overlay`; `capability.rs` `Ui` head list).                                                                      |
| Admission       | One surface per window per generation, admitted through the accepted `OverlayManager` as `OverlayKind::NonModal` on `OverlayTier::Popup`; `modal_active` stays false and one non-modal slot is consumed (`plugin_runtime/presentation_overlay.rs` `show`).                                                                              |
| Second claimant | Another plugin's `show` fails closed with `E_UI_UNAVAILABLE` and leaves the incumbent untouched. The owner may re-show with the same or a replacement `overlay` handle; the newest handle wins on the same slot (`presentation_overlay.rs` `show`).                                                                                     |
| Placement       | Centered over the focused view, clipped to that view's content rect, painted above the focused view and above chrome bands, below the focusable capture surface; a frame narrower than `PRESENTATION_OVERLAY_MIN_COLS` (10) paints nothing and keeps the session live (`runtime/present.rs` paint order, `paint_presentation_overlay`). |
| No input        | No hit-testing and no pointer routing: clicks pass through to the surface beneath, no focus change occurs, and no capture session is created (`runtime/present.rs` `paint_presentation_overlay` docs).                                                                                                                                  |
| Focus-follow    | Binds to the focused view on the first tick after `show`, re-binds when focus moves, and drops the surface with one `view-closed` observation when the bound view is no longer live (`presentation_overlay.rs` `observe_focus`; `PluginRuntime::observe_presentation_focus`).                                                           |
| Frame budget    | `PRESENTATION_OVERLAY_MAX_ROWS` (64) and `PRESENTATION_OVERLAY_MAX_BYTES` (16 KiB) per frame, applied like a chrome band: whole trailing rows are dropped deterministically and the drop is reported through `Runtime::presentation_overlay_paint` (`runtime/present.rs`).                                                              |
| Dismissal       | `Esc` dismisses without consuming the press; a coincident confirmation gate keeps its own consume. The plugin observes one `overlay.dismissed` event on the cold tick (see below) and never runs on the input path (`crates/bitty-terminal/src/terminal_app.rs` `dismiss_presentation_overlay`, `deliver_overlay_dismissed`).           |
| Teardown        | `hide` is idempotent and ungated; suspend, unload, disable, crash, rollback, and generation change drop the surface and release its slot. Lifecycle teardown queues no observation (`presentation_overlay.rs` `revoke_plugin`).                                                                                                         |
| Safe mode       | No plugin generation exists, so no surface exists; a generation that reaches the host under safe policy is refused with `E_UI_UNAVAILABLE` before admission (`services.rs` `require_presentation_overlay`).                                                                                                                             |

The accepted [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
does not name `ui.present`: the head was added by Core after ratification
(bitty#2849, bitty PR #2856) and this page records it as current host fact
without amending the accepted contract, on the same rule as
[Accepted RFC divergence](#accepted-rfc-divergence).

### `bitty.ui.present.show(handle)` errors

Checked in this order at `bitty@8108e4f7` (`host.rs` `present` table →
`services.rs` `ui_overlay_show_with_expiry`):

1. Argument type: a non-integer handle is `E_UI_COMPONENT_INVALID` (class
   `validation`) in the callback before any host call.
2. Call deadline: an expired bounded call is `E_TIMEOUT` (class `budget`) and
   commits nothing.
3. Safe mode: `E_UI_UNAVAILABLE` before the grant check.
4. Grant: a missing `ui.present` is `E_CAPABILITY_DENIED` (class `runtime`),
   naming `ui.present`.
5. Ownership: a handle this generation did not mount into the `overlay` slot is
   `E_UI_NOT_OWNER` (class `runtime`); a previous generation's handle can never
   alias, so this also covers stale handles.
6. Admission: a second plugin, or a host without the shared session, is
   `E_UI_UNAVAILABLE` (class `runtime`).

`bitty.ui.present.hide` raises none of these. It is not gated on `ui.present`,
safe mode, or host wiring, so a surface whose grant was revoked mid-session can
still be removed; a foreign, stale, or repeated handle returns `false` and never
steals another plugin's surface (`services.rs` `ui_overlay_hide`).

### `overlay.dismissed` observation

The presenting plugin receives one `overlay.dismissed` event per dismissal, on
the application cold tick, with payload `{ owner, handle, reason }`: `owner` is
the presenting plugin id, `handle` the presented block handle, and `reason` is
`"escape"` (Core routed `Esc`) or `"view-closed"` (the view the panel was drawn
over closed). Delivery requires the kind in the manifest's `lazy.events`, the
same precondition `bitty.events.subscribe` enforces
(`crates/bitty-runtime/src/plugin_runtime/mod.rs` `deliver_event`;
`ESCAPE_DISMISS_REASON` and `VIEW_CLOSED_DISMISS_REASON` in
`presentation_overlay.rs`). The payload carries
no input bytes, terminal content, or coordinates, and the buffer holds at most
`PENDING_DISMISSED_MAX` (64) pending observations with drop-oldest, so a
show/dismiss loop inside one command callback can lose its earliest ones
(`presentation_overlay.rs` `PENDING_DISMISSED_MAX`, `dismiss_with_reason`).

## Example

Band-hosted mount and update with `ui.rich` granted, verified against
`crates/bitty-runtime/tests/plugin_ui.rs` (`probe_plugin_mounts_bottom_block`,
`lua_band_slot_mounts_land_in_the_expected_band`, `bitty@7d98f53a`):

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
`crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@7d98f53a`):

```lua
-- Manifest grants ui.rich, ui.overlay, and ui.overlay.focus.
local overlay = bitty.ui.mount("overlay", { kind = "Text", text = "pick" })
-- overlay is a retained block_id; it is not routed to a chrome band.
bitty.ui.overlay.acquire(overlay) -- starts the Core-owned capture session
```

Unavailable and removed names fail closed, verified against
`vertical_band_slots_fail_closed_at_mount` and
`tabline_slot_requires_exclusive_claim` in
`crates/bitty-runtime/tests/plugin_ui.rs` and the
`ui_mount_rejects_unknown_slot_at_host_boundary` unit test in
`crates/bitty-runtime/src/plugin_runtime/services.rs` (`bitty@7d98f53a`):

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
(`band_slots.rs:338`) at `bitty@7d98f53a`.

A granted presentation panel draws without capture, verified against
`presentation_overlay_api.rs` (`bitty@8108e4f7`):

```lua
-- Manifest grants ui.rich, ui.overlay, ui.present, and lazy.events = {"overlay.dismissed"}.
local handle = bitty.ui.mount("overlay", { kind = "Column", children = {
  { kind = "Text", text = "which-key", fg = "accent", bold = true },
  { kind = "Text", text = "g  go to line" },
  { kind = "Text", text = "d  delete" },
} })
bitty.ui.present.show(handle) -- true; above the focused view, no focus change

bitty.events.subscribe("overlay.dismissed", function(ev)
  -- ev.reason == "escape" | "view-closed"; Core already removed the surface.
end)
-- Esc dismisses the panel and still reaches the focused terminal untouched.
```

## Limits

- Hosted subset: `top` and `bottom` are painted edge bands, and `overlay` is a
  Core focusable-overlay host that is not a band. `left`, `right`, `tabline`,
  and `terminal` fail with `E_UI_UNAVAILABLE`; `statusline` is not a slot and
  fails with `E_UI_COMPONENT_INVALID` (`band_slots.rs` module docs,
  `bitty@7d98f53a`). All of this is host-build behavior in a `draft` reference
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
- Overlay style parity, typed component validation, band-only interaction
  routing, and per-generation bounds are reconciled in
  [Component shape, style parity, and band interaction](#component-shape-style-parity-and-band-interaction).
- Overlay scene-node pointer routing stays a design note: no accepted overlay
  hit-test contract exists, so declared click/scroll routes run on bands only
  and overlay input goes through the `ui.overlay.focus` capture path. The
  presentation-only overlay is paint-only for the same reason: no hit test, no
  focus, and no `Esc` consumption (`runtime/present.rs`, `bitty@8108e4f7`).
- Per-band per-frame render caps and terminal-wide scene aggregation are not
  implemented in this build; the accepted scene budgets are enforced per
  plugin generation (deferred, not promoted).
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
depend on the removed name. The accepted RFC vocabulary was amended on
2026-10-26 (CTX-1546, bitty issue #2676); the amendment is recorded in
[Accepted contract amendment record](#accepted-contract-amendment-record).

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (accepted; "UI contributions (L2)" spellings, the amended seven-name slot
  set, v1 node set, `block_id` versioning, `ui.rich` / `ui.overlay` /
  `ui.overlay.focus` gates, LUA-OQ-7, LUA-OQ-11). Its pre-2026-10-26
  `statusline` vocabulary and non-focusable overlay wording were amended; see
  [Accepted contract amendment record](#accepted-contract-amendment-record).
- [Plugin Platform RFC](../../specifications/plugin-platform-rfc.md)
  (accepted; restates the amended slot vocabulary).
- [Lua Reference Overview](overview.md) (error contract, capability gating,
  executability rule, L1/L2 split)
- [Lua API Reference](README.md) (route table)
- `bitty` `crates/bitty-runtime/src/runtime/band_slots.rs`
  (`bitty@7d98f53a`, bitty PR #2234, CTX-1341; module docs placement table,
  stacking, exclusive zone, and known gaps; `ui_slot_placement` at
  `band_slots.rs:87`; `unsupported_slot_error` at `band_slots.rs:112`; tests
  `every_v1_slot_has_exactly_one_placement`,
  `from_mounts_counts_unsupported_slots_instead_of_dropping_silently`,
  `unsupported_error_is_typed`)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (`bitty@7d98f53a`; `PluginServices::ui_mount` gate order at
  `services.rs:2800`; `ui_update` at `services.rs:2848`; overlay capture gates
  and `UiBlocks` registry bounds)
- `bitty` `crates/bitty-runtime/tests/plugin_ui.rs` (`bitty@7d98f53a`;
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
- `bitty` `crates/bitty-lua/src/ui.rs` (`bitty@7d98f53a`; `UiSlot` current
  seven-name set, `read_component`, `ui_slot_enum_is_the_closed_v1_set` pins
  the rejection of `statusline`)
- `bitty` `crates/bitty-lua/src/host.rs` (`bitty@7d98f53a`; `ui_mount`
  default at `host.rs:1085`; `ui_update` default at `host.rs:1117`;
  `ui_mount_with_expiry` / `ui_update_with_expiry` pre-commit docs;
  `E_UI_UNAVAILABLE` constant; `bitty.ui.overlay.*` capture contract)
- `bitty` `crates/bitty-plugin-host/src/capability.rs`
  (`CapabilityFamily::Ui`; `ui.rich`, `ui.overlay`, `ui.overlay.focus`,
  `ui.present`, `ui.protocol-register` closed identifiers, the last added by
  bitty#2849 at `bitty@8108e4f7`)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/presentation_overlay.rs`
  (`bitty@8108e4f7`, bitty#2849 / bitty PR #2856; `PresentationOverlay::show`,
  `hide`, `revoke_plugin`, `dismiss_escape`, `observe_focus`;
  `ESCAPE_DISMISS_REASON`, `VIEW_CLOSED_DISMISS_REASON`,
  `PENDING_DISMISSED_MAX`; unit tests
  `show_admits_one_non_modal_popup_surface`,
  `second_plugin_fails_closed_while_owner_resbows`,
  `hide_is_idempotent_and_never_steals`,
  `dismissal_observations_are_bounded_with_drop_oldest`)
- `bitty` `crates/bitty-runtime/src/runtime/present.rs` (`bitty@8108e4f7`;
  paint order above chrome bands and below the capture surface,
  `paint_presentation_overlay`, `PRESENTATION_OVERLAY_MIN_COLS`,
  `PRESENTATION_OVERLAY_MAX_ROWS`, `PRESENTATION_OVERLAY_MAX_BYTES`,
  and the `presentation_overlay_*` paint tests)
- `bitty` `crates/bitty-runtime/tests/presentation_overlay_api.rs` and
  `crates/bitty-lua/tests/ui_bridge.rs` (`bitty@8108e4f7`; presentation
  admission, denial, dismissal, and Lua verb round-trip coverage)
- `bitty` `crates/bitty-lua/tests/ui_bridge.rs`
  (`host_without_ui_surface_fails_closed`,
  `probe_bitty_ui_namespace_is_present`,
  `mount_and_update_round_trip_validated_scene`,
  `capability_denial_propagates_typed`)
- `bitty` band/overlay style parity and interaction routing, synced from issue
  [#2191](https://github.com/bitty-terminal/bitty/issues/2191): PR
  [#2385](https://github.com/bitty-terminal/bitty/pull/2385) adds the parity
  matrix `crates/bitty-runtime/tests/lua_ui_parity.rs`
  (`every_v1_kind_accepts_and_renders`, `property_matrix_drop_one_fails`,
  `unsupported_components_fail_typed`, `bounds_fail_closed`,
  `slot_placement_is_exhaustive`, `click_and_scroll_route_through_commands`,
  `legacy_span_keeps_primary_button_and_args`,
  `regions_are_derived_from_painted_spans`, `hot_path_is_pure_data`,
  `chrome_gestures_keep_terminal_authority`) plus the Lua-VM test
  `lua_component_carries_every_accepted_property` in
  `crates/bitty-lua/tests/ui_bridge.rs`; PR
  [#2392](https://github.com/bitty-terminal/bitty/pull/2392) adds the
  `styled_empty_overlay_row_keeps_pill_chrome` and
  `overlay_root_declaration_reaches_rows` regressions in
  `crates/bitty-runtime/src/runtime/present.rs`
  (`plugin_overlay_paint_tests`).
- Contract tracking: CTX-1299 (bounded composition API design) and CTX-1305
  (release-scope reconciliation) remain open under their own tasks; CTX-1419
  is the completed implementation evidence this page reconciles; the
  accepted-contract amendment is tracked in issue
  [#2676](https://github.com/bitty-terminal/bitty/issues/2676) (not in scope
  for this reference).
- `bitty-plugin-sdk` (issue #154, PR #155; `surface/bitty-plugin-api-v1.json`
  `ui.mount.errors` / `ui.update.errors`; `tests/mock-host.test.ts` timeout
  and block-budget coverage; `conformance/cases/36-ui-error-vocabulary.json`)
