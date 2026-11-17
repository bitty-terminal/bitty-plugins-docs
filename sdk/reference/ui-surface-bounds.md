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
> implemented in `bitty` at `94039dd1`, with the style table re-read and
> re-pinned against `bitty` main at `4bc0fce5`, and its height rows at
> `0a7f1f15`, where the page says so, covering the bounded composition work
> tracked in bitty [#2191](https://github.com/bitty-terminal/bitty/issues/2191),
> [#2411](https://github.com/bitty-terminal/bitty/issues/2411), and
> [#3176](https://github.com/bitty-terminal/bitty/issues/3176). It carries no
> compatibility promise and adds no requirement to the accepted
> [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md). Every
> entry names the file and the symbol or line that enforces it; anything the
> sources at those revisions do not show is not stated here.

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
  (`bitty.ui.overlay.acquire`/`update`/`poll`), deny-by-default
  (`services.rs:2845`); `release` is deliberately ungated so a revoked grant
  can still free input (`services.rs:2845` docs).
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
exactly: ten keys, closed, in the rich reader only. Every unit, default, and
cap below is the bridge's own, re-read from `bitty` main at `4bc0fce5` with the
height rows re-read at `0a7f1f15` (`crates/bitty-lua/src/ui_style.rs`), whose
module doc is the normative primitive table and states the same numbers in
prose. Citations in this section name the symbol rather than a line, because
the table grew in four landed steps and a line pin would have rotted between
them.

| Key            | Unit        | Default              | Inclusive cap              | Pinned by                    |
| -------------- | ----------- | -------------------- | -------------------------- | ---------------------------- |
| `padding`      | cells       | `0`                  | 32                         | `UI_STYLE_MAX_PADDING_CELLS` |
| `radius`       | physical px | `0`                  | 16                         | `UI_STYLE_MAX_RADIUS_PX`     |
| `border`       | physical px | `0`                  | 16                         | `UI_STYLE_MAX_BORDER_PX`     |
| `border_token` | theme token | `content.foreground` | closed six-name set        | `UiBorderToken::ALL`         |
| `min_width`    | cells       | `0` (no floor)       | 1000                       | `UI_STYLE_MAX_WIDTH_CELLS`   |
| `max_width`    | cells       | `0` (no cap)         | 1000                       | `UI_STYLE_MAX_WIDTH_CELLS`   |
| `min_height`   | cells       | `0` (no floor)       | 1000                       | `UI_STYLE_MAX_HEIGHT_CELLS`  |
| `max_height`   | cells       | `0` (no cap)         | 1000                       | `UI_STYLE_MAX_HEIGHT_CELLS`  |
| `gap`          | cells       | `0` (no spacing)     | 32                         | `UI_STYLE_MAX_GAP_CELLS`     |
| `align`        | closed set  | `start`              | `start` / `center` / `end` | `UiStyleAlign::ALL`          |

The closed key set is `UI_STYLE_FIELDS`, in canonical declaration order:
`padding`, `radius`, `border`, `border_token`, `min_width`, `max_width`,
`min_height`, `max_height`, `gap`, `align`. An unlisted name is refused with a
typed `E_UI_COMPONENT_INVALID` naming it, never ignored — `width`, `height`,
`margin`, `spacing`, `align_items`, and `stroke` are among the refused
spellings, because one concept has one name and a second spelling would let one
node declare two answers. A repeated key is refused too, though only a
hand-built marshalled table can carry one: the Lua VM collapses duplicate keys
in a literal before the bridge sees the value. The caps reuse accepted ranges
rather than inventing a ceiling: `padding`, `radius`, and `border` reuse the
accepted decoration bounds, `gap` reuses the same `0..=32` spacing range
`padding` uses, and both size caps reuse the accepted grid-dimension bound —
`UI_STYLE_MAX_WIDTH_CELLS` and `UI_STYLE_MAX_HEIGHT_CELLS` are both `1000`,
because that bound is a property of a grid, not of one axis. One past any cap
is a typed refusal naming the found value and the cap; there is no clamp and no
best-effort default.

The six border tokens are unchanged and still exact: `content.foreground`,
`content.accent`, `content.muted`, `state.error`, `state.warning`, and
`state.success`. A colour literal, a gradient, a Core chrome token (`border.*`,
`chrome.*`), and any other name are rejected.

**The widths bound the content box.** `min_width` and `max_width` are cells on
the width inside `padding`, not on the padded outer frame, so a width
declaration composes with an inset instead of fighting it. `0` is the inert
idiom `border = 0` uses — no constraint on that side — so a floor alone means
"at least this wide" and is never read as `N..0`. Equal bounds are a legal
fixed width. A floor above the declared cap is refused rather than reordered or
clamped (`scene style width range is inverted: min_width N > max_width M`), and
`UiNodeStyle::validate` is the type-level half of the same rule.

**The heights bound the same box vertically.** `min_height` and `max_height`
are the vertical half of that vocabulary and take the same shape: cells on the
content box inside `padding`, `0` meaning no constraint on that side, equal
bounds a legal fixed height, and an inverted range refused instead of reordered
(`scene style height range is inverted: min_height N > max_height M`,
`UiStyleError::HeightRangeInverted`; `UiNodeStyle::validate` is again the
type-level half). The cap is `UI_STYLE_MAX_HEIGHT_CELLS = 1000` — the same
accepted grid-dimension bound the widths reuse, because that bound is a
property of a grid rather than of one axis — so a height one past it is a typed
refusal that reports the value verbatim. Core applies the pair, not the plugin:
`SceneNodeStyle::constrain_height` (`bitty-rich/src/scene.rs:640`) is called
from the measure and paint paths (`crates/bitty-render/src/scene.rs:1335`,
`:1595`, `:1649`). The floor raises the frame the node paints, so a one-line
capsule can claim a three-row pill without Core inventing content; the cap
narrows the rows handed to the child, so content past it is clipped by the
existing walk. A declared floor the allocation cannot fit clips at the
allocation — the fit failure a `min_width` sees in a narrow slot — and never
overflows, wraps, or scrolls. Unlike `gap` and `align`, the pair needs no
composed axis, so it is refused on no node kind: a height is the node's own
extent, not a relation between siblings.

**`gap` is spacing between siblings.** It is the distance from the end of one
sibling's span to the start of the next, on the composed axis, in cells, capped
at 32. `Row`, `Column`, and `List` may declare it; the rich reader refuses a
non-zero `gap` on any other kind with a typed `E_UI_COMPONENT_INVALID` naming
the kind, because a leaf has no adjacent pair to separate. A declared `0` is
accepted everywhere and is flat. A childless or single-child container still
accepts a gap that separates nothing, and an over-subscribed container clips at
the slot boundary: no sibling is truncated to make room, nothing wraps, nothing
scrolls, and the gap is never shrunk. The composed axis belongs to the paint
path, not to the field — a `Column`/`List` composes horizontally in a band and
vertically in an overlay — so a plugin must not read `gap` on a `Column` as a
vertical rhythm inside a band.

**`align` places direct children.** The set is closed at three spellings,
`start`, `center`, and `end`, exact and case-sensitive, on `Row`, `Column`, and
`List`. It places that container's direct children inside its content box on
the axis its siblings do not advance: a `Row` walks left to right, so `align`
moves them vertically, and a `Column`/`List` walks top to bottom, so it moves
them horizontally. `center` sits a child at `slack / 2` and `end` at `slack`,
where `slack` is the content box minus the child's own measured extent, and it
saturates: a child wider than the box has no slack and stays at offset zero. It
moves children inside a box the container already owns, so it changes no
measured size. An absent `align` and a declared `start` are both today's
offset-zero walk and both flat. `space-between` is **not** a value here — it is
the band root `Row`'s four-name distribution on a different surface, and the
two cannot meet on one node, so each refuses the other's exclusive spelling.

**`border_token` is inert without geometry.** The token resolves to an enum, so
a style that declares only a token is flat: it paints nothing and costs nothing
until a width or radius gives it something to colour.

Elevation, shadow, opacity, blur, transitions, transforms, z-order, per-node
fonts, and background images are **not** part of this table. Of the style and
layout vocabulary the candidate records name, `min_width`, `max_width`,
`min_height`, `max_height`, `gap`, and `align` are now **landed** — they are the
six additive rows above, present in `UI_STYLE_FIELDS` and parsed by
`parse_node_style` — while `size` (one
size-constraint key) and `states` (closed interaction states) remain
**candidate and unlanded**: no `style` key parses either, and a declaration
naming one is refused as an unknown field rather than tolerated. The unlanded
half is owned by
[Lua UI Style and Layout Vocabulary (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/lua-ui-style-layout-candidate.md);
the four paint primitives stay owned by
[Scene Style Primitives (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md).
Both are `status: draft`, and neither is a compatibility promise.

Two behaviours shape how the table is used, neither of which is a row in it:

- **One stroke per node.** `style.border` (physical px) and a `Block`'s own
  `border = { width = N }` (cells) are two spellings of one stroke, so a node
  that declares both with a non-zero width is refused with a typed
  `E_UI_COMPONENT_INVALID` naming both widths; a zero width paints nothing and
  therefore never collides (`UiNodeStyle::collides_with_cell_frame`, applied by
  the rich reader's `Block` admission, `bitty-lua/src/ui_rich.rs`; the
  cell-frame cap is 8 cells, `UI_RICH_MAX_BORDER_CELLS`).
- **Loud drop on band and overlay.** The band/overlay reader does not host the
  table: a `style` key there is the one tolerated field it _reports_. The
  reader counts every node that declared one (`UiTolerated::rich_style_nodes`,
  `bitty-lua/src/ui.rs`), and after the mount lands the bridge calls
  `HostServices::ui_style_ignored` once per generation, which emits one fixed
  diagnostic naming the surface and the node count. The mount **succeeds** and
  the tree is unchanged: this is a diagnostic, never a refusal, and never a
  silent drop. Every other unknown optional field on this surface is still
  ignored silently under the accepted additive-minor rule, pinned by
  `unknown_properties_are_tolerated`
  (`crates/bitty-runtime/tests/lua_ui_parity.rs`).

### Backwards compatibility

A style is flat when every geometry value is `0` and `align` is `start`
(`UiNodeStyle::is_flat`); the declared `border_token` is then inert, because it
resolves to an enum and colours nothing on its own. A flat style — the empty
style included — lowers to no wrapper node and emits no paint operation, so an
existing `Text`/`Row`/`Column`/`List` tree that never declares `style` renders
byte for byte as it did before this vocabulary landed. That is the whole
compatibility claim, and it is why every new field defaults to the value that
describes today's rendering: `0` for the two widths, the two heights, and for
`gap`, `start` for `align`.

A non-flat style charges exactly one scene node and one depth level on top of
the content it styles, against the `SCN-1` node budget and the bridge's
`UI_MAX_DEPTH` ceiling, so a tree already at either ceiling is refused rather
than silently wrapped. Nothing else is charged: style values are not text and
cost no `SCN-3` bytes, and neither `border_token` nor `align` retains a string
under `SCN-4` because both resolve to an enum.

**Contract status of the table.** The vocabulary is _landed in code, candidate
in contract_: the paint half is defined by
[Scene Style Primitives (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md)
and the additive half by
[Lua UI Style and Layout Vocabulary (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/lua-ui-style-layout-candidate.md),
both `status: draft`, and it is not named by the accepted Plugin API v1 Lua
Surface RFC. The bridge module states the same thing in its own header. Do not
read this table as an accepted amendment; read it as the current behaviour of
`bitty@4bc0fce5`, with the height rows at `0a7f1f15`, under two candidate
vocabularies. The later rows landed in separate pull requests on `bitty` main:
bitty [#3186](https://github.com/bitty-terminal/bitty/pull/3186) (`9ebe5285`,
`min_width` and `max_width`), bitty
[#3188](https://github.com/bitty-terminal/bitty/pull/3188) (`90deb814`, `gap`),
bitty [#3198](https://github.com/bitty-terminal/bitty/pull/3198) (`9f407a37`,
`style.align`), and the `min_height`/`max_height` pair for bitty
[#2191](https://github.com/bitty-terminal/bitty/issues/2191), present at
`0a7f1f15`.

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
- [Lua UI Style and Layout Vocabulary (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/lua-ui-style-layout-candidate.md)
  (draft candidate; the two width bounds, the height pair, `gap`, and `align`
  landed; `size` and `states` unlanded)
- `bitty` `crates/bitty-lua/src/ui.rs` (`ui.rs:182` slots, `:208`/`:211` node
  sets, `:214`-`:232` budgets, `:308`/`:322` hover phase and queue, `:331`
  anchor, `:531` tolerated declarations, `:922` excluded-kind rejection,
  `:1181` hover region requirement)
- `bitty` `crates/bitty-lua/src/ui_style.rs` (`UI_STYLE_FIELDS` closed ten-key
  set, `UI_STYLE_MAX_*` caps, `UiNodeStyle::is_flat`, `UiNodeStyle::validate`,
  `UiNodeStyle::collides_with_cell_frame`, `parse_node_style`; module doc owns
  the primitive table and the accounting)
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
