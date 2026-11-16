---
title: bitty.ui Surface Bounds and Capability Boundary
description: Landed closed vocabularies capability map contract budgets style table and exclusions an SDK consumer designs against
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 131
---

# bitty.ui Surface Bounds and Capability Boundary

> Status: **draft**. This page records the closed vocabularies, the capability
> map, and the contract ceilings of the declarative `bitty.ui` surface as
> implemented in `bitty` at `94039dd1`, covering the bounded composition work
> tracked in bitty [#2191](https://github.com/bitty-terminal/bitty/issues/2191),
> [#2411](https://github.com/bitty-terminal/bitty/issues/2411), and
> [#3176](https://github.com/bitty-terminal/bitty/issues/3176). It carries no
> compatibility promise and adds no requirement to the accepted
> [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md). Every
> entry names the file and line that enforces it; anything the sources at that
> revision do not show is not stated here.

## Purpose and scope

A plugin author needs one answer to three questions before writing a scene:
what may I declare, which grant does the declaration need, and which ceiling is
enforced when it is admitted. This page records those three, plus the boundary
of what the surface deliberately cannot express. It is a surface-and-bounds
record, not a second call reference.

Canonical ownership stays where it already is, and this page does not restate
it:

- The accepted contract — the slot vocabulary, the `block_id` handle model,
  the L1/L2 split, and the exclusions — is the
  [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  ("UI contributions (L2)" and the v1 rejection table).
- The call contract (`mount`/`update` behaviour, handles, and the error table)
  and slot-to-host placement are the [bitty.ui reference](ui.md).
- Cross-repository contract reconciliation is owned by
  [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs):
  [Lua UI Composition Reconciliation](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/lua-ui-composition-reconciliation.md),
  [Chrome Surface API (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/chrome-surface-api-candidate.md),
  and [Scene Style Primitives (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md)
  (all `status: draft` there). Those records own the contract wording; this page
  records what the `bitty` bridge and host actually enforce, with evidence.

Out of scope: per-slot host placement, the error-code table, the overlay
capture and presentation sessions, and the capability lifecycle. Each is linked
above or in [References](#references) instead of duplicated here.

## Closed slot set and capability map

The bridge's slot enum is the single source of truth for slot names: seven
names, frozen for v1 — `terminal`, `top`, `bottom`, `left`, `right`, `tabline`,
`overlay` (`UI_SLOTS`, `bitty` `crates/bitty-lua/src/ui.rs:182`). The Lua
bridge and the host mount gate parse through the same `UiSlot::parse`, so a
name is either in the closed set or rejected with `E_UI_COMPONENT_INVALID` in
the bridge, before any host call (`ui.rs:1454` pins the enum against
`UI_SLOTS`; the host re-checks at the boundary, `crates/bitty-runtime/src/plugin_runtime/services.rs:6602`).

| Slot       | Grant required at mount                                                  | Enforced at                                       |
| ---------- | ------------------------------------------------------------------------ | ------------------------------------------------- |
| `terminal` | `ui.rich`                                                                | `services.rs:6685` (rich scene path)              |
| `top`      | `ui.rich`                                                                | `services.rs:6608`                                |
| `bottom`   | `ui.rich`                                                                | `services.rs:6608`                                |
| `left`     | `ui.rich`                                                                | `services.rs:6608`                                |
| `right`    | `ui.rich`                                                                | `services.rs:6608`                                |
| `tabline`  | `ui.rich` plus an exclusive `[lazy].claims` entry naming `workspaceline` | `services.rs:6608`, claim gate `services.rs:6634` |
| `overlay`  | `ui.rich` **and** `ui.overlay`                                           | `services.rs:6608` and `services.rs:6611`         |

Three more heads belong to the same capability family and are deliberately
separate grants, none implied by another (`crates/bitty-plugin-host/src/capability.rs:152-161`):

- `ui.overlay.focus` — the transient input-capture verbs
  (`bitty.ui.overlay.acquire`/`update`/`release`/`poll`), deny-by-default
  (`services.rs:2845`).
- `ui.present` — presenting an already-mounted `overlay` block without capture
  (`bitty.ui.present.show`/`hide`), deny-by-default (`services.rs:2878`).
- `ui.protocol-register` — protocol registration, unrelated to scene
  composition and named here only so the family is complete
  (`capability.rs:161`).

The map is a grant map, not a placement map: every accepted slot needs
`ui.rich` even when the build presents nothing there, because the capability
gate runs before the placement gate (`services.rs:6608` then `:6649`). An
author therefore cannot infer "this slot is hosted" from "this slot is
accepted", and a build that starts presenting a slot later does not change
which grant that slot needs.

## Node vocabulary

The band and overlay reader accepts exactly four kinds — `Text`, `Row`,
`Column`, `List` — and rejects five by name — `Block`, `Image`, `CodeBlock`,
`Table`, `Rule` — with `E_UI_COMPONENT_INVALID`
(`UI_V1_NODE_KINDS`, `bitty-lua/src/ui.rs:208`;
`UI_V1_EXCLUDED_NODE_KINDS`, `ui.rs:211`; the rejection, `ui.rs:922`). The
accepted RFC excludes the same five (`../plugin-api-v1-lua-surface-rfc.md`,
"UI contributions (L2)").

The exclusion is a property of the _band/overlay reader_, not of the bridge.
The `terminal` slot is read by a second, richer reader
(`read_rich_component`, `bitty-lua/src/ui_rich.rs:463`) whose vocabulary is the
nine accepted `SceneNode` kinds (`UI_RICH_NODE_KINDS`, `ui_rich.rs:119`), and
the bridge routes `bitty.ui.mount("terminal", …)` there
(`bitty-lua/src/host.rs:6485`). Two consequences an author must design around:

- A node kind valid on `terminal` is not valid on `top`/`bottom`/`left`/
  `right`/`overlay`, and the reverse holds for the presentation fields: `fg`,
  `bg`, `anchor`, `on_click`, `on_scroll`, `on_hover`, and `region` are refused
  on the rich reader with `E_UI_COMPONENT_INVALID` rather than dropped
  (`ui_rich.rs:470-482`).
- The two readers do not share a style vocabulary; see
  [Landed style table](#landed-style-table).

## Contract budgets

These are the accepted numbers, not bridge inventions. Each row names the
contract that owns the value, the constant that pins it, and where it fails.
The per-block accounting detail and the handle contract stay in the
[bitty.ui reference](ui.md#per-generation-bounds).

| Budget                                         | Contract value | Owning contract                                                                                                                                      | Pinned at                                                                               | Failure, where                                                                              |
| ---------------------------------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Nodes per block (`SCN-1`)                      | 2048           | [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md) (accepted, `SCN-1`) | `UI_MAX_NODES`, `ui.rs:217`, checked `ui.rs:889`                                        | `E_UI_COMPONENT_INVALID`, bridge                                                            |
| Tree depth per block                           | 16             | Bridge budget, deliberately stricter than accepted `SCN-2 = 32`                                                                                      | `UI_MAX_DEPTH`, `ui.rs:214`, checked `ui.rs:910`                                        | `E_UI_COMPONENT_INVALID`, bridge                                                            |
| Text bytes per block (`SCN-3`)                 | 256 KiB        | Rich Presentation RFC, `SCN-3`                                                                                                                       | `UI_MAX_TEXT_BYTES`, `ui.rs:220`, checked `ui.rs:895`                                   | `E_UI_COMPONENT_INVALID`, bridge                                                            |
| Retained blocks per generation (`SCN-5`)       | 64             | Rich Presentation RFC, `SCN-5` (a per-terminal number, enforced per generation as a strict subset)                                                   | `UI_MAX_BLOCKS`, `ui.rs:224`, `services.rs:1230`                                        | `E_UI_BLOCK_BUDGET`, host registry                                                          |
| Retained string bytes per generation (`SCN-4`) | 2 MiB          | Rich Presentation RFC, `SCN-4` (same per-generation subset)                                                                                          | `UI_MAX_AGGREGATED_TEXT_BYTES`, `ui.rs:232`, `services.rs:1239`                         | `E_UI_BLOCK_BUDGET`, host registry                                                          |
| Pending hover dispatches per frame             | 8              | Host budget for band pointer routing (bitty #2411)                                                                                                   | `UI_HOVER_QUEUE_MAX`, `ui.rs:322`, `crates/bitty-runtime/src/runtime/band_host.rs:1658` | Newest transition dropped fail-closed, counted in `hover_queue_drops` (`band_host.rs:3159`) |

Two accounting facts the numbers alone do not convey:

- The `SCN-4` aggregate is measured over every _retained_ string, not only
  `Text` content: theme tokens, command identifiers, argument keys and values
  are charged too (`ui.rs:93-97`). The 2 MiB number is unchanged; the
  measurement is a strict superset of text-only accounting.
- Terminal-wide aggregation across plugins is not implemented. `SCN-4`/`SCN-5`
  are per-terminal in the accepted contract; this bridge enforces the same
  numbers per plugin generation, which is a strict subset (`ui.rs:88-91`).

Binding bounds, all fail-closed at the bridge (`bitty-lua/src/ui.rs`):

| Field                          | Bound     | Constant                         |
| ------------------------------ | --------- | -------------------------------- |
| Command identifier             | 64 bytes  | `UI_CLICK_MAX_COMMAND_BYTES:235` |
| Declared arguments per binding | 16        | `UI_CLICK_MAX_ARGS:238`          |
| Argument value                 | 256 bytes | `UI_CLICK_MAX_ARG_BYTES:241`     |
| Argument key                   | 64 bytes  | `UI_CLICK_MAX_ARG_KEY_BYTES:245` |
| Theme token name               | 32 bytes  | `UI_MAX_THEME_TOKEN_BYTES:253`   |
| Region identity                | 64 bytes  | `UI_MAX_REGION_ID_BYTES:262`     |

Host-injected arguments are reserved out of the 16-entry cap rather than added
to it: the bridge caps a declaration at `UI_CLICK_MAX_ARGS` minus the host keys
it does not already name (`ui.rs:1281`), so a region-aware binding may declare
at most 14 (click reserves `region` and `button`) or 15 (scroll reserves
`region` and `delta`), and an unbound click may declare all 16. The host merges
its own keys last, overwriting a declared entry of the same name rather than
duplicating it (`band_click_args_table`, `band_host.rs:1147`;
`band_hover_args_table`, `band_host.rs:1388`).

## Landed style table

The style table is the boundary case of this whole page, so it is stated
exactly. Four keys, closed, in the rich reader only:

| Key            | Unit        | Default              | Inclusive cap       | Evidence              |
| -------------- | ----------- | -------------------- | ------------------- | --------------------- |
| `padding`      | cells       | `0`                  | 32                  | `ui_style.rs:99`      |
| `radius`       | physical px | `0`                  | 16                  | `ui_style.rs:104`     |
| `border`       | physical px | `0`                  | 16                  | `ui_style.rs:109`     |
| `border_token` | theme token | `content.foreground` | closed six-name set | `ui_style.rs:118-163` |

The closed key set is `UI_STYLE_FIELDS` (`bitty-lua/src/ui_style.rs:94`). The
caps reuse accepted decoration ranges rather than inventing a ceiling
(`ui_style.rs:5-15`). The six border tokens are `content.foreground`,
`content.accent`, `content.muted`, `state.error`, `state.warning`, and
`state.success`; a colour literal, a gradient, a Core chrome token
(`border.*`, `chrome.*`), and any other name are rejected (`ui_style.rs:46-50`).
Elevation, shadow, opacity, blur, transitions, transforms, z-order, per-node
fonts, background images, and the candidate `gap`/`align`/`size`/`states`
vocabulary are **not** part of this table (`ui_style.rs:35-37`).

Four rules an author must know:

- **One stroke per node.** `style.border` (physical px) and a `Block`'s own
  `border = { width = N }` (cells) are two spellings of one stroke, so a node
  that declares both with a non-zero width is refused with a typed
  `E_UI_COMPONENT_INVALID` naming both widths; a zero width paints nothing and
  therefore never collides (`ui_style.rs:53-63`; enforcement
  `bitty-lua/src/ui_rich.rs:358-378`; the cell-frame cap is 8,
  `ui_rich.rs:113`).
- **No-op when flat.** An all-zero style lowers to no scene node and paints
  nothing, so an unstyled tree renders byte for byte as before
  (`ui_style.rs:64-66`).
- **It costs nodes.** A non-flat style is a wrapper node, so it charges one
  node and one depth level against `SCN-1`/`SCN-2`; a tree already at the
  ceiling is refused rather than silently wrapped (`ui_style.rs:68-78`). Style
  values charge no `SCN-3` text bytes, and `border_token` retains no string
  under `SCN-4` because it resolves to an enum (`ui_style.rs:70-72`).
- **Loud drop on band and overlay.** The band/overlay reader does not host the
  table: a `style` key there is the one tolerated field it _reports_. The
  reader counts every node that declared one (`UiTolerated`, `ui.rs:531`;
  counting at `ui.rs:930`), and after the mount lands the bridge calls
  `HostServices::ui_style_ignored` once per generation, which emits one fixed
  diagnostic naming the surface and the node count
  (`host.rs:3750-3758` → `services.rs:6754`). The mount **succeeds** and the
  tree is unchanged: this is a diagnostic, never a refusal, and never a silent
  drop. Every other unknown optional field on this surface is still ignored
  silently under the accepted additive-minor rule, pinned by
  `unknown_properties_are_tolerated`
  (`crates/bitty-runtime/tests/lua_ui_parity.rs:581`).

**Contract status of the table.** The vocabulary is _landed in code, candidate
in contract_: it is defined by
[Scene Style Primitives (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md)
(`status: draft`) and is not named by the accepted Plugin API v1 Lua Surface
RFC. The bridge module states the same thing in its own header
(`ui_style.rs:5-15`). Do not read this table as an accepted amendment; read it
as the current behaviour of `bitty@94039dd1` under a candidate vocabulary.

## Declarative command bindings

Interaction is data. A node may carry `on_click`, `on_scroll`, and `on_hover`
bindings shaped `{ command, args, region? }`, plus an optional per-item
`region = { id, kind = "item" | "module" }` identity; no field accepts a
function (`ui.rs:11-20`, region kind at `ui.rs:1343`). The host injects its own
keys last, overwriting any declared entry of the same name: `region` and
`button` for a click, `region` and `delta` for a scroll, `region` and `phase`
for a hover (`ui.rs:269`, `:275`, `:291`; hover table assembly,
`crates/bitty-runtime/src/runtime/band_host.rs:1388-1408`).

- **Hover requires a region identity.** `on_hover` is rejected at the bridge
  when no `region` is declared, because a hover without a stable id would have
  nothing to key a transition on (`ui.rs:1181-1191`). `phase` is `"enter"` or
  `"leave"` (`ui.rs:308-314`). `on_click`/`on_scroll` may carry a region; a
  node that declares regions on both must declare the same one (`ui.rs:946`).
- **No coordinates reach the plugin.** A dispatch carries the owning plugin,
  the declared command, the declared arguments, the stable region id, and a
  bounded host value (`delta`, `button`, `phase`) — never a pointer position
  (`ui.rs:40-41`; `BandHoverRequest`, `band_host.rs:1546-1556`). Region
  rectangles are derived from the same painted spans that paint and hit-test
  use, and clip to the window
  (`regions_are_derived_from_painted_spans`, `lua_ui_parity.rs:845`).
- **No plugin VM on the render or input path.** The band host resolves a
  binding and queues a request; the application drains it and dispatches the
  registered command through the normal command path
  (`drain_band_clicks`/`drain_band_scrolls`/`drain_band_hovers`,
  `band_host.rs:3177`; pinned by `hot_path_is_pure_data`, `lua_ui_parity.rs:869`).
  Pointer motion therefore costs one bounded hit test, not a Lua call.
- **Hover never captures input and never takes modal authority.** Motion is
  presentation-only: it never captures input, never sets `modal_active`, and is
  inert while the window is unfocused or while a focusable overlay capture
  holds, so a modal can never re-arm a tooltip underneath itself
  (`band_host.rs:1551-1554`, `:3043-3046`).
- **The queue coalesces, then drops.** At most one transition per changed
  region is queued per motion; the drain returns at most 8 per frame and an
  overflow drops the newest transition fail-closed while counting a diagnostic,
  so a dropped edge can never wedge state (`band_host.rs:3122-3164`).
- **Routing is band-only.** Declared bindings dispatch from band surfaces; the
  bridge module states that overlay click/scroll/hover routing is not part of
  the accepted contract and remains a design note (`ui.rs:44-46`). Overlay
  input goes through the separately gated capture session instead (see
  [bitty.ui reference](ui.md#capability)).

### Group anchors

A direct child of a scene's root `Row` may declare
`anchor = "left" | "center" | "right"`; the band host resolves it against the
presented row width (`UiAnchor`, `ui.rs:331`; resolution
`flatten_band_runs_in_width`, `band_host.rs:359`). The spelling set is closed
and case-sensitive (`ui.rs:355-360`). An `anchor` anywhere else — on the root,
below the root's direct children, or in a scene that has no bounded row — is
refused with `E_UI_COMPONENT_INVALID` rather than silently ignored
(`ui.rs:1197-1215`, scope enum `ui.rs:380`). Only `top`/`bottom` validate as a
band scope; a `left`/`right` column band and the `overlay` slot validate as
unanchored (`services.rs:6739-6752`).

## Not in v1

Each item is absent from the surface at `bitty@94039dd1`, not merely
undocumented here.

- **Panel registration.** No `register_panel`, `PanelId`, `PanelProvider`, or
  panel lifecycle on this surface; v1 rejects panel providers
  (`../plugin-api-v1-lua-surface-rfc.md:171` and `:640`). The `bitty.panel.*`
  namespace in `bitty` is a separate, post-v1 surface with its own
  `panel.create`/`panel.focus` grants
  (`crates/bitty-lua/src/host.rs:2836-2862`) and does not route through
  `bitty.ui`.
- **Level 3 presentation.** Decorations, annotations, and semantic
  highlighting belong to Level 3 in
  [Plugin system](../../extensibility/plugin-system.md), not to the L2 scene
  contribution.
- **Level 4 protocol extension.** OSC/APC and structured-output registration is
  Level 4 in the same document; `ui.protocol-register` is a capability head,
  not a scene field.
- **Renderer-side objects.** No renderer handles, shaders, pipelines, glyph
  injection, native windows, or global coordinates
  (`../plugin-api-v1-lua-surface-rfc.md`, "UI contributions (L2)").

## Compatibility and versioning

Draft reference: this page states current bridge and host behaviour at
`bitty@94039dd1` and carries no compatibility promise while its status is
draft. The compatibility target is the
[Compatibility policy](../plugin-api-v1-lua-surface-rfc.md#compatibility-policy)
of the accepted RFC: additions ship as minor versions, removals or narrowings
require a major version.

Three divergences are visible at this pin and are recorded rather than
smoothed over:

1. **Style table.** Landed in code, candidate in contract; see
   [Landed style table](#landed-style-table). Promotion into the accepted
   surface is an owner amendment, not a docs change.
2. **`terminal` and `left`/`right` are hosted at this pin.** The accepted rule
   is that a slot a build does not present fails closed; _which_ slots a build
   presents is host policy, not contract. At `94039dd1` the bridge routes a
   `terminal` mount to the rich reader (`bitty-lua/src/host.rs:6485`), and the
   placement policy maps `left`/`right` to column bands
   (`crates/bitty-runtime/src/runtime/band_slots.rs:161-162`) while keeping
   `terminal` out of band placement because it attaches a scene instead
   (`band_slots.rs:172`). The placement table in the [bitty.ui reference](ui.md)
   is pinned to an earlier revision and records the earlier build; reconciling
   it is a docs task in this repository, not a contract change.
3. **Placement drift is not capability drift.** Placement is not recorded here
   on purpose: the [bitty.ui reference](ui.md) owns it, and slot-to-host
   placement is being reconciled in `bitty-terminal-docs`. The capability map
   above is unaffected by placement changes, because the grant gate runs first
   (`services.rs:6608`).

Any change to the capability map, a budget value, or the style key set is a
contract change and must update this page, the
[bitty.ui reference](ui.md), and the owning contract together.

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (accepted; slot set, L2 contribution surface, exclusions, panel-provider
  rejection, compatibility policy)
- [bitty.ui reference](ui.md) (call contract, placement, handles, error table,
  overlay capture and presentation)
- [Lua API Reference](README.md) (route table for this tree)
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  (accepted; `SceneNode`, `SCN-1`..`SCN-5`)
- [Lua UI Composition Reconciliation](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/lua-ui-composition-reconciliation.md)
  and [Chrome Surface API (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/chrome-surface-api-candidate.md)
  (draft candidates; cross-repository contract reconciliation)
- [Scene Style Primitives (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md)
  (draft candidate; `padding`, `radius`, `border`, `border_token`)
- `bitty` `crates/bitty-lua/src/ui.rs` (`ui.rs:182` slots, `:208`/`:211` node
  sets, `:214`-`:232` budgets, `:308`/`:322` hover phase and queue, `:331`
  anchor, `:531` tolerated declarations, `:922` excluded-kind rejection,
  `:1181` hover region requirement)
- `bitty` `crates/bitty-lua/src/ui_style.rs` (`:94` closed key set, `:99`-`:115`
  caps, `:53` one stroke per node, `:68` accounting)
- `bitty` `crates/bitty-lua/src/ui_rich.rs` (`:8` terminal-slot reader,
  `:119` rich kinds, `:358` one-stroke refusal, `:470` band-field rejection)
- `bitty` `crates/bitty-lua/src/host.rs` (`:6485` terminal-slot mount routing,
  `:3750` once-per-generation style notice)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/services.rs` (`:6608`
  capability gates, `:6634` tabline claim, `:6649` placement gate, `:1125`
  block registry, `:1230`/`:1239` registry bounds, `:6754` style diagnostic)
- `bitty` `crates/bitty-runtime/src/runtime/band_host.rs` (`:359` anchored runs,
  `:1551` hover carries no authority, `:1658` hover queue bound, `:3122` queue
  overflow, `:3177` drains)
- `bitty` `crates/bitty-runtime/tests/lua_ui_parity.rs` (`:164` every v1 kind,
  `:331` unsupported kinds fail typed, `:581` unknown properties tolerated,
  `:624` bounds fail closed, `:744` click/scroll routing, `:845` region
  derivation, `:869` hot path is pure data)
