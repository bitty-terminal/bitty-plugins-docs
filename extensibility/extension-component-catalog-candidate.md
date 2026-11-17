---
title: Extension Component Catalog (Candidate)
description: Candidate catalog of reusable extension Scene composition patterns with scene shape style and width hooks slot capability caller evidence and current-build availability
category: extensibility
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 31
---

# Extension Component Catalog (Candidate)

> Status: **draft candidate** — not **Accepted**, not **Verified**, not
> **Compatible**, and not normative. This record catalogues how downstream
> extensions already compose the shipped declarative surface into five reusable
> patterns: an edge band, a pill row, a tooltip and detail fallback, a modal
> sheet, and a status strip. Every entry names the plugin and file that
> hand-rolls the pattern today, so the catalog describes shipped compositions
> rather than inventing an API.
>
> Nothing here adds a node kind, slot, style field, interaction binding,
> capability, budget, or error code, and nothing here relaxes the accepted
> [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md).
> Where a pattern needs something the current build does not carry, the entry
> says so and is marked a **gap** instead of being sketched as if it worked.
>
> This revision extends all five entries with the hooks that landed in the style
> table after the four paint primitives — `min_width` and `max_width`, the
> `min_height` / `max_height` pair (all four capped at `1000` content cells
> each), `gap`, the sibling spacing (capped at `32` cells), and `align`, the
> child placement — ten keys in total — and with a per-entry **contract**
> block: the closed-set slot the pattern mounts into, the four v1 node kinds it
> composes, the bridge budget it charges, and the command route its interaction
> takes.
> Each entry ends with a proof status that separates _bridge-verified_ (the field
> parses and paints, and `bitty` code proves it) from _caller-demonstrated_ (a
> shipped plugin already does it) from _unverified_ (no artifact in this
> repository proves the
> behaviour). The honest summary of the width work is negative and is stated
> once, plainly, in [Width hooks](#width-hooks-what-the-landed-table-does-not-unlock):
> the width hooks are rich-reader-only, so they do not reach any of the five
> patterns on the surface each one belongs to, and no shipped plugin declares
> them.
>
> No entry claims headed verification, because none exists: this revision
> produced no screenshot, no run log, and no headed lane. The examples are
> headed listings — complete scenes with their manifest grant — and they are
> illustrations, not test results.
>
> This record **supersedes** the earlier
> [Plugin UI Component Catalog](ui-component-catalog-candidate.md), which was
> written against the surface before the bounded `style` table, band group
> anchors, and pointer-hover routing shipped, and which recorded no
> caller-level evidence. The earlier record is retained as a deprecated
> pointer only.

## Purpose and scope

Plugins keep rebuilding the same five surfaces by hand: an edge band, a row of
status capsules, a detail hint, a transient sheet, a status strip. Each one
re-derives the scene tree, the theme tokens, and the interaction bindings, and
each lands on slightly different answers. The useful cases already exist as
shipped code in seven plugins, but that shape is discoverable only by reading
their sources.

This catalog turns that into a documented, contract-shaped record: five named
patterns (E-1 to E-5), each with the scene tree it mounts, the style hooks that
apply to it on the surface it belongs to, the slot and capability it needs, the
caller that hand-rolls it today, its accessibility position, the bounds it must
respect, and a headed example.

In scope:

- E-1 edge band, E-2 pill row, E-3 tooltip and detail fallback, E-4 modal
  sheet, E-5 status strip.
- For every entry: scene shape, style hooks (including which width hooks reach
  the entry's surface), slot and capability, caller evidence at `file:line`,
  accessibility notes, bounds, a contract block, and a headed Lua example.
- A per-entry proof status: what is bridge-verified, what a shipped caller
  demonstrates, and what is `unverified` or a `gap`.
- A per-entry availability verdict against the **current build**: **shipped**,
  **partial**, or **gap**.
- The explicit residual list, including the one deliverable this record does not
  and cannot satisfy from a documentation repository: the bar plugin is not yet
  migrated onto these entries.

Out of scope and owned elsewhere:

- the closed slot set, the `SceneNode` subset, the handle model, and the
  capability gates (accepted,
  [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md);
  current host facts in the [bitty.ui reference](../sdk/reference/ui.md));
- per-slot purpose, multiplicity, bounds, and conflict resolution (candidate,
  [Plugin UI Slot Inventory](plugin-ui-slot-inventory-candidate.md));
- the five-level component hierarchy, the composition principle, and the
  accessibility direction (candidate,
  [Lua UI Component Model](lua-ui-component-model-candidate.md));
- the style primitive set, its caps, and its resolution order (candidate,
  [Scene Style Primitives](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md));
- token naming and theme resolution (candidate,
  [Theme Token Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/theme-token-contract-candidate.md));
- host band geometry, stacking, clipping, and paint order (terminal-side; the
  [bitty.ui reference](../sdk/reference/ui.md) states the host facts);
- panel identity, activity stacks, focus routing, and the single Core-owned
  modal authority (accepted,
  [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md));
  `register_panel` and `PanelProvider` stay excluded from plugin API v1;
- any shared recipe package or service (candidate,
  [Cross-Plugin Composition](cross-plugin-composition-candidate.md)).

## Normative sources this specification must not weaken

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (accepted; amended 2026-10-26): the seven-name closed slot vocabulary, the v1
  node subset (`Text`, `Row`, `Column`, `List`), the opaque
  generation-owned `block_id`, the `ui.rich` / `ui.overlay` / `ui.overlay.focus`
  / `ui.present` gates, the exclusive `tabline` claim, and the exclusion of
  `register_panel`.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  (accepted): `SceneNode`, `RichBlock` replacement, and scene bounds
  `SCN-1`..`SCN-5` — 2048 nodes, depth 32, 256 KiB text per block, 2 MiB
  aggregated rich bytes, 64 blocks per terminal.
- [Scene Style Primitives](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md)
  (candidate, shipped as a bridge slice): the closed style table, its units, its
  inclusive caps `32` / `16` / `16` / `1000` / `1000` / `1000` / `1000` / `32`,
  its six-name closed content-token set, and its per-field resolution order. The
  candidate text closes the _paint_ set at four primitives
  (`scene-style-primitives-candidate.md:145`), while the bridge parses ten keys
  (`UI_STYLE_FIELDS: [&str; 10]`,
  `crates/bitty-lua/src/ui_style.rs:279-290` at `origin/main` `0a7f1f15`): the
  two width bounds landed at `9ebe5285`, the sibling `gap` at `90deb814`,
  `align` at `9f407a37`, and the height pair for bitty
  [#2191](https://github.com/bitty-terminal/bitty/issues/2191), all ahead of
  that candidate text, which now records them as landed behaviour rather than
  as proposals of its own (`ui_style.rs:3-15`, `:46-90`). This catalog describes
  the bridge, and says which of the two a statement comes from.
- [bitty.ui reference](../sdk/reference/ui.md) (draft; current host fact at its
  pinned revision): the band token table, the `bold` double strike, the
  `on_click` / `on_scroll` binding shape with `region` identities, band-only
  dispatch, overlay style parity, overlay capture, presentation-only overlay,
  and the per-generation bounds the bridge enforces.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  (accepted): panel identity, focus routing, and the single Core-owned modal
  authority no catalog entry may transfer to a plugin.
- [Plugin system](plugin-system.md) (draft): plugins alter presentation, never
  Terminal Truth; capabilities are deny-by-default; composition goes through
  declared host surfaces.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (accepted):
  per-plugin resource ceilings no pattern may widen.
- [Security Overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md)
  and [Threat Model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md):
  presentation-not-truth, no hot-path execution, bounded inputs.

If an entry here contradicts a normative source, the normative text wins. This
catalog has no authority to promote a candidate into a contract, and it cannot
widen a bound it quotes.

## Terminology

| Term            | Meaning in this document                                                                                                                                                                                                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Pattern         | A named composition of shipped scene nodes, style hooks, and bindings. Documentation, not API.                                                                                                                                                                                             |
| Scene shape     | The node tree a pattern mounts, written with the four v1 kinds.                                                                                                                                                                                                                            |
| Band vocabulary | The four-kind declarative surface `bitty.ui.mount` accepts for every band and the overlay: `Text`, `Row`, `Column`, `List`.                                                                                                                                                                |
| Rich vocabulary | The nine-kind declarative surface the `terminal` slot reads, which is the only surface that accepts a `style` table.                                                                                                                                                                       |
| Style hook      | A field the host paints: `fg`, `bg`, `bold` on the band vocabulary; `style = { padding, radius, border, border_token }` on the rich vocabulary.                                                                                                                                            |
| Token           | A host-resolved name. Band colour tokens are `foreground`, `background`, `cursor`, `selection`, `accent`. Style border tokens are the six `content.*` / `state.*` names. Never a raw colour.                                                                                               |
| Group           | A `Row` child of a band root carrying `anchor = "left" \| "center" \| "right"`.                                                                                                                                                                                                            |
| Atom            | One `Text` leaf — the only node that carries visible characters.                                                                                                                                                                                                                           |
| Caller evidence | The shipped plugin file and line that hand-rolls the pattern today.                                                                                                                                                                                                                        |
| Availability    | **Shipped** (usable with shipped spellings), **partial** (one rung works, another does not exist), **gap** (not expressible; no API invented here), **unverified** (landed in the bridge, but no shipped caller or artifact proves the behaviour the entry describes).                     |
| Proof status    | The evidence level an entry reaches: _bridge-verified_ (a field parses and paints, shown at `bitty` `file:line`), _caller-demonstrated_ (a shipped plugin already hand-rolls it, shown at plugin `file:line`), or **unverified** / **gap** as above. No entry reaches headed verification. |
| Composition     | The entry's node tree written with the four v1 kinds only. Not a Rust type, not a constructor, not a capability.                                                                                                                                                                           |

Every entry carries the same seven fields: a contract block (slot, nodes,
style/width hooks, interaction route, budget), scene shape, style hooks, slot
and capability, caller evidence, accessibility notes, and bounds, plus a headed
example and a proof status. Availability is stated first, because a pattern that
needs a missing primitive is not a pattern.

## Two vocabularies, one catalog

The single most common source of wrong advice about this surface is treating
"the declarative UI" as one thing. Two readers accept two different field sets,
and a field accepted by one is either ignored or rejected by the other. The
catalog entries are written against this table.

| Field                                                                                                         | Band vocabulary (`top`, `bottom`, `left`, `right`, `overlay`)                                                                 | Rich vocabulary (`terminal`)                                                                                             |
| ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `Text`, `Row`, `Column`, `List`                                                                               | Accepted                                                                                                                      | Accepted                                                                                                                 |
| `Block`, `Image`, `CodeBlock`, `Table`, `Rule`                                                                | Rejected (`E_UI_COMPONENT_INVALID`)                                                                                           | Accepted                                                                                                                 |
| `fg` / `bg` theme tokens                                                                                      | Accepted; closed five-name table, unknown well-formed names fall back and are counted                                         | **Rejected**                                                                                                             |
| `bold`                                                                                                        | Accepted on every kind                                                                                                        | Accepted on `Text`                                                                                                       |
| `style = { padding, radius, border, border_token, min_width, max_width, min_height, max_height, gap, align }` | Not part of the vocabulary; an unknown optional key this build tolerates, does **not** paint, and reports once per generation | Accepted; caps `32` / `16` / `16` / `1000` / `1000` / `1000` / `1000` / `32`; six-name closed token set                  |
| `min_width` / `max_width` / `min_height` / `max_height` (inside `style`)                                      | Not painted; a band keeps the width and the height Core gives it                                                              | Accepted; bounds the **content box** in cells on either axis, `0` = unbounded on that side, an inverted range is refused |
| `gap` (inside `style`)                                                                                        | Not painted; a band keeps the spacing its flatten walk produces                                                               | Accepted; separates adjacent siblings in `Row`/`Column`/`List`, `0` = absent, refused on `Text`                          |
| `anchor` on a direct child of the root                                                                        | Accepted in a band scope; rejected on the root, below it, or in a non-band scope                                              | **Rejected**                                                                                                             |
| `on_click` / `on_scroll` / `on_hover`                                                                         | Accepted; dispatched from bands only (overlay routing is not shipped: `ui.rs:46-49`)                                          | **Rejected**                                                                                                             |

Consequences a pattern author has to hold:

1. A band cannot be rounded, padded, bordered, or sized today. Declaring `style`
   on a band node is inert: the band path keeps the accepted additive-minor
   tolerance for unknown optional fields, so the key neither paints nor errors —
   and the host now says so once per generation, naming the surface and the
   number of nodes that declared it (`services.rs:6754`).
2. A rich node cannot be theme-filled. `style` carries a border colour, not a
   fill, and `bg` is rejected there — so "rounded capsule with an accent fill"
   is not expressible on either surface.
3. Interaction lives on the band side only. A rich scene is content; a band is
   chrome that can be clicked, scrolled, and hovered.
4. `style` costs one node and one depth level per styled node, because a
   non-flat declaration lowers to a wrapper. A tree at the node or depth ceiling
   that declares a style is refused, not admitted over budget.
5. Width is a rich-reader concept. `min_width` and `max_width` bound the content
   box in cells and are applied by Core's measure path
   (`bitty-rich/src/scene.rs:619`); the band and overlay readers drop them with
   the rest of the table. The same holds for sibling spacing: `gap` is an integer
   cell count on the same rich reader, capped at `32`, refused on a `Text` leaf
   because a leaf has no siblings, and dropped by a band or overlay mount with
   the rest of the table. No pattern in this catalog is a rich-reader pattern,
   so no pattern gains a width or a spacing it can set — see
   [Width hooks](#width-hooks-what-the-landed-table-does-not-unlock).

## Entry contract: the bounds every entry inherits

Seven properties hold for every entry below. Each is pinned in `bitty` code at
`origin/main` (`90deb814`) rather than inherited from prose, and an entry that
appears to break one is a documentation defect, not an extension.

**Slots: the closed seven.** `terminal`, `top`, `bottom`, `left`, `right`,
`tabline`, `overlay` (`UI_SLOTS`, `bitty-lua/src/ui.rs:182`, generated from
`UiSlot::ALL` at `ui.rs:142` so the enum, the string list, and the parser cannot
disagree). `statusline` is **retired**: it is not one of the seven, `ui.mount`
rejects the name with `E_UI_COMPONENT_INVALID` before any host call, and a test
pins the rejection alongside `Statusline`, `status`, `workspaceline`, and `tabs`
(`ui.rs:1455-1470`). It appears in this document only as the retired spelling.
Three placement facts decide which entry may claim which slot:

- `top` / `bottom` are one-row bands, and they are the only surfaces where a
  declared `anchor` resolves: the scope selector maps `Top | Bottom` to
  `UiSceneScope::Band` and everything else to `Unanchored`
  (`services.rs:6749`; scope enum `ui.rs:380-386`).
- `left` / `right` are **column** bands: the root must be column-shaped
  (`Column` or `List`, or an axis-neutral `Text`), so a `Row`-rooted entry — E-1
  and E-5 — cannot mount there at all, and an `anchor` inside one is refused
  because the surface is `Unanchored` (`band_slots.rs:36-38`, `:241-259`;
  `services.rs:6745-6748`). A column band's width is the widest direct child of
  its root in display cells, so the scene sets the band thickness even though the
  band cannot carry `style`.
- `tabline` is an exclusive claim over Core tab-projection state and has no host
  in this build: a mount fails `E_UI_UNAVAILABLE` and the slot is
  `NeverPresented` (`band_slots.rs:163-165`, `:226`). No entry uses it.

**Nodes: four kinds.** `Text`, `Row`, `Column`, `List`
(`UI_V1_NODE_KINDS`, `ui.rs:208`). `Block`, `Image`, `CodeBlock`, `Table`, and
`Rule` are v1-excluded (`UI_V1_EXCLUDED_NODE_KINDS`, `ui.rs:211`) and refused by
name (`ui.rs:922`). The `terminal` slot is read by a second reader that accepts
all nine kinds (`ui_rich.rs:113`, `:119`), but no entry here uses the extra
five: an entry states a composition in the intersection, so the same shape means
the same thing whichever reader evaluates it.

**Style: ten keys, rich reader only.** `padding` (cells, cap 32), `radius`
(physical px, cap 16), `border` (physical px, cap 16), `border_token` (closed
six-name content set), `min_width` and `max_width` (cells, cap 1000 each), the
`min_height` / `max_height` pair (the same content-box range on the vertical
axis, cap 1000 each), `gap` (cells, cap 32), and `align` (closed three-name
placement) — `UI_STYLE_FIELDS: [&str; 10]`, `ui_style.rs:279-290` at
`origin/main` `0a7f1f15`; caps `ui_style.rs:295`, `:300`, `:305`, `:320`,
`:329`, `:340`. `0` means "no constraint" on a width or a height side and "no
spacing" on `gap`, and a floor above the cap is refused rather than reordered
(`ui_style.rs:879` for the width range, `:902` for the height range). Width
bounds the **content box** inside `padding`, and Core applies it in the measure
path (`SceneNodeStyle::constrain_width`, `bitty-rich/src/scene.rs:619`): the
plugin declares a range and never runs layout. The height pair bounds the same
box on the other axis, applied by `SceneNodeStyle::constrain_height`
(`bitty-rich/src/scene.rs:640`) from the `bitty-render` measure and paint paths:
a floor raises the frame the node paints, a cap narrows the rows handed to the
child, an inverted pair is refused rather than swapped (`scene style height
range is inverted: min_height N > max_height M`), and a floor the allocation
cannot fit clips at the allocation instead of overflowing. Unlike `gap` and
`align` it needs no composed axis, so it is refused on no node kind: a height is
the node's own extent, not a relation between siblings. `gap` separates
adjacent siblings inside a `Row`, `Column`, or `List` and advances the composed
walk, so a following sibling starts `gap` cells later; a declared `0` renders
identically to an absent key, a non-zero value on a kind that composes no
siblings is refused `E_UI_COMPONENT_INVALID` rather than dropped
(`ui_rich.rs:460`), and a group that does not fit is clipped, never wrapped,
scrolled, or shrunk.

**Budgets: five numbers, per generation.** 2048 nodes (`UI_MAX_NODES`,
`ui.rs:217`), depth 16 (`UI_MAX_DEPTH`, `ui.rs:214`, deliberately stricter than
accepted `SCN-2 = 32`), 256 KiB text (`UI_MAX_TEXT_BYTES`, `ui.rs:220`), 64
retained blocks (`UI_MAX_BLOCKS`, `ui.rs:224`), 2 MiB retained strings
(`UI_MAX_AGGREGATED_TEXT_BYTES`, `ui.rs:232`). A non-flat style — padding,
radius, border, or either width — is a wrapper node, so it charges one node and
one depth level on top of what it styles (`ui_style.rs:93-105`), and a tree at a
ceiling is refused rather than wrapped.

**Interaction: registered commands only.** `on_click`, `on_scroll`, and
`on_hover` are data shaped `{ command, args, region? }`; no field accepts a
function, no render or input hot path enters the plugin VM, and no entry
declares or implies a capability (`ui.rs:11-20`; style parsing is a pure
cold-path function, `ui_style.rs:85-90`). Hover requires a `region` and receives
`phase`, never coordinates (`UI_HOVER_PHASE_ARG`, `ui.rs:308`). Colours are
theme tokens and never literals; neither reader carries a shader, image, path, or
renderer handle, and no node reads terminal text.

**An entry is composition guidance, not a class.** Nothing here is a Rust type,
a constructor, a capability, or an error code. An entry names a shape an author
writes in Lua from the four kinds plus the style table, and names the shipped
caller that proves the shape is in use today.

**Examples are not verification.** Every entry ends with a headed listing — a
complete scene plus the manifest grant and command registration it needs, written
to be run as one file. None has been executed against a headed host for this
revision: there is no screenshot, no run log, and no headed lane anywhere in this
repository. An example is an illustration, and no entry may be cited as headed
verification.

## Catalog

| ID  | Pattern                   | Scene shape                                                   | Style hooks                                                                              | Slot(s)                    | Required capability                                                                      | Hand-rolled today                                                                                                                                                           | Availability                                                                              |
| --- | ------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | -------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| E-1 | Edge band                 | `Row` root, up to three anchored `Row` groups of `Text` atoms | `fg`, `bg`, `bold` on atoms and groups; no padding, radius, border, or width             | `top`, `bottom`            | `ui.rich`                                                                                | `bar/lua/bar/init.lua:4336`, `:4474`, `:4746`; `file-manager/lua/file-manager/init.lua:38`, `:222`; `devtools/lua/devtools/view.lua:45`, `:400`                             | **Shipped** (band vocabulary)                                                             |
| E-2 | Pill row                  | `Row` of padded `Text` capsules inside a band group           | `fg`, `bg`, `bold`; radius and width unavailable on the band path                        | `top`, `bottom`            | `ui.rich`                                                                                | `bar/lua/bar/init.lua:104`-`:114`, `:1945`, `:1966`-`:1990`                                                                                                                 | **Shipped** as square capsules; rounded pill is a **gap**                                 |
| E-3 | Tooltip / detail fallback | Band `Text` with `on_hover` + one retained overlay `Text`     | Band fields on the trigger; same token table on the overlay tooltip; no `style` or width | `top`, `bottom`, `overlay` | `ui.rich` + `ui.overlay`                                                                 | `bar/lua/bar/init.lua:1576`, `:4701`-`:4745`, `:4964`-`:5000`                                                                                                               | **Partial**: inline, hover-overlay, and notify rungs shipped; anchored popup is a **gap** |
| E-4 | Modal sheet               | `Column` of `Text` rows in the overlay (no backdrop, no trap) | Band fields only; non-modal by contract; no `style` or width                             | `overlay`                  | `ui.rich` + `ui.overlay` (+ `ui.overlay.focus` for keys, or `ui.present` for paint-only) | `git-panel/lua/git-panel/init.lua:680`, `:689`; `history/lua/history/init.lua:809`, `:840`; `palette/lua/palette/init.lua:225`, `:230`; `help/lua/help/init.lua:470`-`:498` | **Gap** for modality; the non-modal substitutes are shipped                               |
| E-5 | Status strip              | `Row` root, anchored groups of module `Text` atoms            | `fg`, `bg`, `bold` per module; single-space join, no separators; no width                | `bottom`                   | `ui.rich` + provider read caps                                                           | `bar/lua/bar/init.lua:141`-`:158`, `:4453`-`:4479`, `:2168`, `:2808`                                                                                                        | **Shipped** (band vocabulary)                                                             |

### Entry composition register

The same five entries written as the two facts a contract review needs: which of
the four v1 kinds each one composes, and whether the width hooks reach the
surface it belongs to.

| ID  | Node composition (v1 kinds only)                   | Composed depth | Slots, from the closed seven     | Width hooks reach it                                                        | Proof status                                                                        |
| --- | -------------------------------------------------- | -------------- | -------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| E-1 | `Row` root → `Row`(anchor) groups → `Text` atoms   | 3              | `top`, `bottom`                  | No: the band reader drops `style`                                           | bridge-verified + caller-demonstrated; width **gap**                                |
| E-2 | `Row` group → `Text` capsule, `Text` separators    | 3              | `top`, `bottom` (via E-1 or E-5) | No on a band; the rich reader accepts `min_width` but rejects the `bg` fill | caller-demonstrated as a square fill; rounded or uniform filled pill **gap**        |
| E-3 | band `Text` trigger + one `overlay` `Text`         | 1 per surface  | `top`, `bottom`, `overlay`       | No: the overlay drops `style`; rich-path text shaping **unverified**        | caller-demonstrated rungs 1-3; anchored popup **gap**                               |
| E-4 | `Column` → `Text` rows                             | 2              | `overlay`                        | No: the overlay drops `style`                                               | caller-demonstrated as capture / paint-only substitutes; modality and width **gap** |
| E-5 | `Row` root → `Row`(anchor) groups → `Text` modules | 3              | `bottom`                         | No: the band reader drops `style`                                           | bridge-verified + caller-demonstrated; width **gap**                                |

Read the fourth and fifth columns together: no entry's surface accepts the width
hooks, so a width statement appears in an entry only to say what does not apply.
The reason is structural, not incidental — every entry here lives on a band, the
overlay, or both, and the rich reader that hosts `min_width`/`max_width` is the
`terminal` slot's reader.

No entry uses `left`, `right`, or `tabline`: the first two require a
column-shaped root, and the third has no host. No entry uses `Block`, `Image`,
`CodeBlock`, `Table`, or `Rule`, all of which are v1-excluded.

## E-1 Edge band

**Availability: shipped on the band vocabulary.** Mounts into `top` or
`bottom` under `ui.rich`.

**Contract.** Slot: `top` or `bottom`, two of the closed seven. Not `left` or
`right` — both are column bands and refuse a row-shaped root
(`band_slots.rs:36-38`, `:241-259`) — and not `tabline`, which has no host.
Nodes: `Row` root → up to three anchored `Row` groups → `Text` atoms, three
levels, nothing else. Style hooks: `fg`, `bg`, `bold` on the band reader. Width:
**not reachable** — a `style` declaration on this surface is dropped and reported
once per generation (`services.rs:6754`), so a band's thickness and width stay
Core's decision. Interaction: `on_click`, `on_scroll`, `on_hover`, each with a
`region` identity and a registered command. Budget: one block, 3 of the 16 depth
levels, one atom per item. Proof: bridge-verified and caller-demonstrated for the
band fields; width is a **gap**.

**Scene shape.** One `Row` root with up to three anchored groups, each a `Row`
of atoms, emitted only when non-empty:

```text
Row                                   root: no anchor
├── Row  anchor = "left"    →  Text…  leading inset
├── Row  anchor = "center"  →  Text…  centred on the full row width
└── Row  anchor = "right"   →  Text…  trailing inset
```

Rules the shape depends on:

- `anchor` is accepted only on a direct child of the root, and only in a band
  scope. On the root itself, below the root's direct children, or in a
  non-band scene it fails closed with `E_UI_COMPONENT_INVALID` rather than being
  silently ignored.
- A child without an `anchor` is a `left` group, so an unanchored band keeps its
  left-to-right flow byte for byte.
- `center` is centred on the full row width, never "after left" — which is why
  the reference bar emits no separator between groups.
- A band whose text is empty is not painted by the host, so an omitted group and
  an empty group are indistinguishable on screen; omit rather than pad.
- Row bands flatten a `Column`/`List` root left to right. Column-shaped bands are
  a separate host placement with their own root-shape rule and are not a band
  group composition.

**Style hooks.** `fg`, `bg`, `bold` on atoms and on group containers. A
container's declaration reaches descendant spans that do not declare their own,
so the group is the broadest lever and the atom's own declaration wins. There is
no padding, radius, border, gap, width, height, or alignment field on this
vocabulary, and the band host owns thickness, insets, stacking, and paint order.

**Slot and capability.** `top` or `bottom`; `ui.rich`. Two plugins on one edge
stack in plugin id byte order, so a pattern never encodes a position.

**Caller evidence.** `bar/lua/bar/init.lua` builds the reference band:
`group_node` at `:4336` returns `{ kind = "Row", anchor = anchor, children = … }`,
`render_bar` at `:4453` assembles the root and omits empty groups (`:4467`-
`:4472`), and `:4746` mounts the result on the configured edge. Two other
plugins mount the same shape with a different content budget:
`file-manager/lua/file-manager/init.lua:38` (`M.SLOT = "bottom"`) with the mount
at `:222`, and `devtools/lua/devtools/view.lua:45` with the mount at `:400`.

**Accessibility notes.** A band receives no keyboard or IME input, so every band
item is pointer-reachable only: an entry needs a command or keymap path for the
same action, and the command id — not the painted label — is what a help surface
documents. The visible text is the only accessible name this surface can carry;
there is no `role`, `label`, `description`, or state field, and the accessibility
property schema remains candidate and owner-pending (U-9 in
[Lua UI Component Model](lua-ui-component-model-candidate.md)). Do not claim that
a band contributes to an accessibility tree.

**Bounds.** One retained block per band. The root, a group, and an atom are
three of the sixteen bridge levels; the per-generation ceilings are 2048 nodes,
depth 16 (`UI_MAX_DEPTH`, deliberately stricter than `SCN-2 = 32`), 256 KiB
text, 64 blocks, and 2 MiB retained strings. The host does not report band
thickness or available width, so every label carries an explicit character
budget instead of assuming space (the reference caps a workspace name at 32
characters and a band line at 1024 bytes).

### Example: edge band

```lua
-- Manifest grants ui.rich.
local function group(anchor, atoms)
  if #atoms == 0 then
    return nil -- omit, never pad an empty group
  end
  return { kind = "Row", anchor = anchor, children = atoms }
end

local function band(cpu)
  local children = {}
  local left = group("left", {
    {
      kind = "Text",
      text = " 1 ",
      fg = "background",
      bg = "accent",
      bold = true,
      on_click = {
        command = "example-plugin:focus",
        args = { workspace = 1 },
        region = { id = "workspace.1", kind = "item" },
      },
    },
  })
  local right = group("right", {
    { kind = "Text", text = "cpu " .. cpu },
  })
  if left ~= nil then
    children[#children + 1] = left
  end
  if right ~= nil then
    children[#children + 1] = right
  end
  return { kind = "Row", children = children }
end

local handle = bitty.ui.mount("bottom", band("12%"))
bitty.ui.update(handle, band("9%")) -- true: the subtree is replaced, the handle is not
```

## E-2 Pill row

**Availability: shipped as square capsules.** Radius is unavailable on the band
path, so the shipped form is a padded square fill.

**Contract.** Slot: inside E-1 or E-5, so `top` or `bottom` — not `left`/`right`
(column root required) and not `tabline`. Nodes: `Row` group → `Text` capsule,
with single-space `Text` separators; there is no `Capsule`, `Button`, `Badge`, or
`Tab` kind in the v1 set (`ui.rs:208`). Style hooks: `fg`, `bg`, `bold`. Width:
**not reachable on a band**, and on the rich reader `min_width` arrives without a
fill — `bg` is rejected there (`ui_rich.rs:70-75`) — so equal-width, filled pills
remain a **gap** on both surfaces. Interaction: `on_click` / `on_hover` with a
per-capsule `region`. Budget: one atom per capsule and no per-pill block; a
region-aware binding reserves `region` and `button`, leaving at most 14 declared
arguments of the 16-entry cap. Proof: caller-demonstrated as a square padded
fill; the rounded or width-equalised filled pill is a **gap**.

**Scene shape.** Each pill is one `Text` atom whose visible string is
`" " .. label .. marks .. " "`. The padding belongs to the same atom as the
fill, so the capsule is one run and one hit target:

```text
Row
├── Text  " 1* "   fg=background  bg=accent  bold=true   on_click=…
├── Text  " "      (the reference separator atom, non-interactive)
└── Text  " 2+! "  bg=selection  fg=cursor                on_click=…
```

There is no `Capsule`, `Button`, `Badge`, or `Tab` node. A capsule is a styled
`Text` atom; a dedicated kind would be a v2+ surface change. Marks append after
the label in a fixed order in every label mode, so a reader can parse one
position deterministically.

**Style hooks.** Fill via `bg`, label via `fg`, emphasis via `bold`. The
reference implementation uses `background` on `accent` for the active capsule,
`selection` for inactive capsules, `bold` for the active and occupied states,
and `cursor` for attention — with attention winning over occupied. Those are
token choices available to any pattern, not reserved names.

Two hard limits follow from the surface, and both are load-bearing:

- The band vocabulary has no radius, so a pill is a square padded fill. A
  rounded pill needs the band path to consume the `style` table, which it does
  not do today: `style` on a band node is an unknown optional key that the
  bridge tolerates and the band paint path never reads.
- On the rich vocabulary, where `padding`, `radius`, `border`, and the two width
  bounds do work, `fg`/`bg` are rejected. A rounded, bordered, width-equalised
  capsule with a token fill therefore needs a fill token that the style table
  does not have — a gap, not a recipe.

**Slot and capability.** Composes inside E-1 or E-5 (`top`, `bottom`);
`ui.rich`.

**Caller evidence.** `bar/lua/bar/init.lua`: the capsule padding constants at
`:104`-`:114` with the surface comment at `:99`-`:103`, the capsule segment
shape at `:1945` (`M.CAPSULE_PAD .. label .. M.CAPSULE_PAD`), and the atom
conversion at `:1966`-`:1990` where marks, fill, and the per-capsule hit region
are attached.

**Accessibility notes.** A pill's accessible text is its visible label plus
marks. If marks encode state, that state must also be reachable through a
command result or a notification, because the surface has no state property. Do
not encode selection in colour alone: the reference active capsule is both
differently filled and bold, which survives a monochrome theme. Pointer-only
reachability means the same action needs a keymap or command entry.

**Bounds.** One atom per pill and no per-pill block; a row of tags that grows
without bound is a node-budget failure, so the pattern caps the visible count
and carries the remainder in a single overflow atom (the reference caps the
workspace strip at 16 capsules). A region-aware binding reserves `region` and
`button`, so at most 14 declared arguments remain of the 16-entry cap.

### Example: pill row

```lua
-- Manifest grants ui.rich and declares the focus and detail commands.
local ACTIVE_FG, ACTIVE_BG, INACTIVE_BG = "background", "accent", "selection"

local function pill(label, active, marks, id)
  return {
    kind = "Text",
    text = " " .. label .. (marks or "") .. " ",
    fg = active and ACTIVE_FG or nil,
    bg = active and ACTIVE_BG or INACTIVE_BG,
    bold = active or nil,
    on_click = {
      command = "example-plugin:focus",
      args = { id = id },
      region = { id = "workspace." .. id, kind = "item" },
    },
    on_hover = {
      command = "example-plugin:detail",
      args = { id = id },
      region = { id = "workspace." .. id, kind = "item" },
    },
  }
end

local handle = bitty.ui.mount("bottom", {
  kind = "Row",
  children = {
    { kind = "Row", anchor = "left", children = {
      pill("1", true, "*", 1),
      { kind = "Text", text = " " },
      pill("2", false, "+!", 2),
    } },
  },
})
```

## E-3 Tooltip and detail fallback

**Availability: partial.** Three rungs ship; the rung an author usually means
by "tooltip" — a popup anchored against its trigger — does not exist.

**Contract.** Slots: `top` or `bottom` for the trigger, `overlay` for the
tooltip — the only three of the seven this entry may name, and `overlay` needs
`ui.overlay` in addition to `ui.rich`. Nodes: one band `Text` trigger plus one
retained overlay `Text` (or `Column` of `Text` rows for a taller card); no richer
kind is used. Style hooks: band fields on both surfaces, because the band and
overlay hosts share one paint path. Width: **not reachable on either surface** —
the overlay drops `style` like the band does (`services.rs:6754`). On the rich
reader `max_width` would bound a tooltip body, but no shipped tooltip lives on
that surface, and whether a narrowed content box wraps, clips, or overflows is
**unverified**: `constrain_width` sizes the box (`bitty-rich/src/scene.rs:619`)
and no artifact this catalog can cite pins the shaping inside it. Interaction:
`on_hover` dispatches a registered command with a `region` and an injected
`phase`. Proof: caller-demonstrated for rungs 1-3; the anchored popup is a
**gap**; rich-path wrapping is **unverified**.

**Scene shape.** A fallback ladder, in preference order:

| Rung | Surface                                                                      | Availability                                                          |
| ---- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| 1    | An inline suffix inside the trigger's own band atom                          | Shipped (band vocabulary)                                             |
| 2    | `on_hover` on the trigger region driving one retained `overlay` `Text` block | Shipped (band vocabulary + `ui.overlay`)                              |
| 3    | `bitty.notify.show { title, body?, urgency? }`                               | Shipped (host-rendered, `platform.notify`)                            |
| 4    | A popup positioned against the trigger's rectangle                           | **Gap**: no anchoring, offset, placement, or per-item geometry exists |

Rung 2 in scene terms — the trigger and the tooltip are two blocks on two
surfaces, joined only by a region id:

```text
band    : Text (on_hover = { command, region = { id = "module:cpu", kind = "module" } })
overlay : Text                       one retained block, replaced on update
```

**Style hooks.** The trigger carries the band fields. The tooltip block is
itself a band-vocabulary node and paints through the same closed token table and
the same bold double strike as a band row, because the band and overlay hosts
share one style path. `style` is not applied on the overlay either, so a rounded
or bordered tooltip is not expressible. The notification rung has no style hooks
at all: rendering, urgency, rate policy, and delivery are host-side.

**Interaction contract.** `on_hover` has the same shape as `on_click` and
**requires** a `region` identity. The host dispatches the declared command on
pointer enter and leave with `phase = "enter"` or `phase = "leave"` injected
beside the `region` id; no coordinates ever reach the plugin, so a tooltip is
anchored by Core-owned region geometry, never by pointer math. Hover never
captures input and never sets a modal state. At most eight band hover dispatches
queue per frame, so a pattern coalesces transitions instead of assuming one
dispatch per painted item.

**Slot and capability.** Band slot (`top`, `bottom`) for the trigger under
`ui.rich`; `overlay` for the tooltip under `ui.rich` + `ui.overlay`. Rung 3
needs `platform.notify`.

**Caller evidence.** `bar/lua/bar/init.lua`: `hover_binding` at `:1576` builds
the region-bound binding, `:4701`-`:4745` owns the hover state, the retained
overlay mount (`:4754`) and the bounded tooltip text update (`:4726`), and
`:4964`-`:5000` handles the enter/leave phases. The same plugin ships both lower
rungs: `atom_node` at `:4313`-`:4320` binds the `detail` command on an atom that
has tooltip text but no click action, and the `detail` command at `:4937`-
`:4960` presents that text as a notification when the hover surface is not the
right channel — the plugin's own words: "explicit fallback; hover uses the
overlay tooltip".

**Accessibility notes.** A hover tooltip is pointer-only, transient, and not
attached to its trigger for assistive technology: there is no hover focus
target, no `description` field, and no role. The accessibility story is
redundancy — the same detail must also be reachable through a command, a
notification, or inline text. Treat dismissal as normal, not as an error: the
presentation-only overlay is dismissed by `Esc` without consuming the press.

**Bounds.** The tooltip is one retained block with bounded text; the reference
collapses whitespace, replaces control bytes with spaces, and cuts the detail at
256 UTF-8 bytes on a character boundary. The overlay block counts against the
same 2048-node / depth-16 / 256 KiB budgets as any other block, and the
notification rung is host rate-limited — a declined or rate-limited show returns
`false`, which the pattern treats as "not shown".

### Example: hover tooltip

```lua
-- Manifest grants ui.rich and ui.overlay; platform.notify for the last rung.
local DETAIL = "example-plugin:detail"
local tooltip -- retained overlay block, mounted once per generation

-- Plugin-owned, bounded hover text per region id.
local DETAILS = { ["module:cpu"] = "cpu 12% · 4 cores · 61 C" }

local function mount_tooltip()
  local ok, handle = pcall(bitty.ui.mount, "overlay", { kind = "Text", text = "" })
  if ok and type(handle) == "number" then
    tooltip = handle
  end
end

local function paint_tooltip(text)
  if tooltip == nil then
    return false -- no overlay grant or no overlay host: the band still paints
  end
  return bitty.ui.update(tooltip, { kind = "Text", text = text or "" }) == true
end

bitty.commands.register({
  id = "hover",
  args_schema = {
    type = "object",
    properties = {
      region = { type = "string" },
      phase = { type = "string", enum = { "enter", "leave" } },
    },
    required = { "region", "phase" },
    additionalProperties = false,
  },
  run = function(args)
    if args.phase == "leave" then
      return paint_tooltip("")
    end
    local detail = DETAILS[args.region] -- bounded, plugin-owned table
    if detail == nil then
      return paint_tooltip("")
    end
    return paint_tooltip(detail)
  end,
})

mount_tooltip()
bitty.ui.mount("bottom", {
  kind = "Row",
  children = {
    {
      kind = "Text",
      text = "cpu 12%",
      on_hover = {
        command = "example-plugin:hover",
        region = { id = "module:cpu", kind = "module" },
      },
      on_click = {
        command = DETAIL,
        args = { module = "cpu" },
        region = { id = "module:cpu", kind = "module" },
      },
    },
  },
})
```

## E-4 Modal sheet

**Availability: gap.** A modal sheet implies a backdrop, a focus trap, an owning
dismissal rule, and a stacking tier above everything else. No v1 node, slot,
field, or call carries any of the four, and the single modal authority stays
Core-owned. This entry records the absence and the non-modal substitutes; it is
not a design for a plugin modal.

**Contract.** Slot: `overlay`, one of the seven, presented only while the
generation owns the transient capture or an active presentation session
(`band_slots.rs:226`). Nodes: `Column` → `Text` rows. Style hooks: band fields
only. Width: **not reachable** — `max_width` binds a node on the rich reader,
which is the `terminal` slot, and `terminal` is content rather than a modal
surface; the overlay drops `style` altogether. A width-stable sheet is therefore
a **gap**, like every other modal property. Interaction: keys arrive only through
the capture session under `ui.overlay.focus`; no callback of the pattern's runs
on the input path. Proof: caller-demonstrated for the capture and paint-only
substitutes; modality, backdrop, focus trap, and width are **gap**.

**Scene shape.** The nearest shipped shape is a `Column` of `Text` rows on the
overlay, with no scrim and no trap:

```text
overlay : Column
├── Text  "Delete 3 files?"          bold
├── Text  "git status  ·  main"
└── Text  "y confirm   n cancel   esc leaves"
```

**Style hooks.** Band-vocabulary fields only. A sheet cannot be rounded or
bordered, because the overlay path does not consume `style`. The rich
vocabulary, which does consume it, is the `terminal` slot's content path — not a
modal surface.

**Slot and capability.** `overlay`. Three substitutes exist, with different
authority:

| Substitute                                     | Grants                                        | What it cannot do                                                       |
| ---------------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------------- |
| Capture overlay (`bitty.ui.overlay.*`)         | the single transient input capture            | no backdrop, no focus trap, no tier selection, no announcement          |
| Presentation overlay (`bitty.ui.present.show`) | draw above the focused view, consume no input | no clicks, no keys, no focus; Core owns `Esc` and reports the dismissal |
| Panel (`register_panel`, `PanelProvider`)      | a host-managed panel with activities          | excluded from plugin API v1                                             |

So a confirm-or-cancel flow either uses the capture overlay and accepts that the
user can leave it with `Esc`, or moves the decision into the terminal, which
already owns input. It must not paint a scrim, claim modality, or block input
outside its own capture session.

**Caller evidence.** The capture path: `git-panel/lua/git-panel/init.lua:680`
(`overlay.acquire { title = … }`) and `:689` (`overlay.update`), with the release
at `:660`; `history/lua/history/init.lua:809` and `:840`;
`palette/lua/palette/init.lua:225` and `:230`; the poll loop at
`palette/lua/palette/init.lua:480`. The paint-only path:
`help/lua/help/init.lua:470`-`:498` (`present.show` / `present.hide`).

**Accessibility notes.** The capture overlay receives keyboard input and is the
only plugin surface that can, which makes it the only place a multi-step flow is
usable without a pointer. It still has no focus trap and no role metadata, so a
destructive confirmation must not rely on modality for safety: require an
explicit typed answer, and treat `Esc` as "leave", never as "confirm". There is
no backdrop and no modal announcement channel.

**Bounds.** One transient capture session per window per generation; a second
claimant fails closed with `E_UI_UNAVAILABLE` while the incumbent keeps the
surface. Capture is single-owner, gated by `ui.overlay.focus`, and revocable by
Core on idle timeout, focus change, unload, or crash; the session end arrives as
the `overlay.released` observation, so a pattern must clear local state from
that event instead of assuming its own release succeeded. The mounted scene
respects the same block and text budgets as any other block.

### Example: bounded confirm without modality

```lua
-- Manifest grants ui.rich, ui.overlay, and ui.overlay.focus.
local session

local function close(reason)
  if session ~= nil and bitty.ui.overlay ~= nil then
    pcall(bitty.ui.overlay.release, session, reason or "cancelled")
  end
  session = nil
end

local function confirm_sheet(question, subject)
  local acquired, handle = pcall(bitty.ui.overlay.acquire, { title = "Confirm" })
  if not acquired or type(handle) ~= "number" then
    return false -- no overlay host or grant: fail closed, do not fake a sheet
  end
  session = handle
  local painted = bitty.ui.overlay.update(session, {
    kind = "Column",
    children = {
      { kind = "Text", text = question, bold = true },
      { kind = "Text", text = subject },
      { kind = "Text", text = "y confirm   n cancel   esc leaves" },
    },
  })
  if painted ~= true then
    close("cancelled")
    return false
  end
  return true
end

-- Core ends the session on Esc, idle timeout, focus change, or unload.
bitty.events.subscribe("overlay.released", function()
  session = nil
end)

local function drain()
  if session == nil then
    return
  end
  local ok, result = pcall(bitty.ui.overlay.poll, session)
  if not ok or type(result) ~= "table" then
    close("cancelled")
    return
  end
  if result.status == "released" then
    session = nil -- Core already closed it; nothing to confirm
    return
  end
  -- handle result.events here; Esc never arrives as a confirmation
end
```

## E-5 Status strip

**Availability: shipped on the band vocabulary.** Mounts into `bottom` under
`ui.rich`. There is no status slot: `statusline` was retired from the slot
vocabulary, and status presentation is a `bottom`-band composition.

**Contract.** Slot: `bottom`, one of the seven. Not `left` or `right` — both want
a column-shaped root, and this entry's root is a `Row` — and not `tabline`, which
has no host. Nodes: `Row` root → anchored `Row` groups → `Text` module atoms
joined by single-space `Text` separators, three levels. Style hooks: per-module
`fg`, `bg`, `bold` on the band reader. Width: **not reachable** — a module's
width is its text and the band's width is Core's; a `style` declaration here is
dropped and reported once per generation (`services.rs:6754`). Interaction: a
clickable module binds a registered command with a `region`; a read-only module
binds nothing rather than binding a command that will be denied. Budget: one
block, one atom per module, 3 of the 16 depth levels. Proof: bridge-verified and
caller-demonstrated; width is a **gap**.

**Scene shape.** A `Row` root of module atoms in anchored groups. Modules are
independent: one that cannot render emits no atom and no placeholder, and only
that module disappears.

```text
Row
├── Row  anchor = "left"    →  Text " 1* "  Text " "  Text " 2 "   (capsules, see E-2)
├── Row  anchor = "center"  →  Text "12:04"
└── Row  anchor = "right"   →  Text "cpu 12%"  Text " "  Text "mem 41%"
```

Modules join with a single literal space inside a group, and no separator is
emitted between groups, because the groups are placed by Core anchors and a
manual pad would break true centring and right flush.

**Style hooks.** Per-module `fg`, `bg`, `bold`. No per-module container styling,
no separators, no dividers: the strip is atoms and single spaces. A pattern may
emphasise one module with `bold` and one state with an accent fill, but must not
encode availability in colour alone.

**Slot and capability.** `bottom`; `ui.rich`, plus the provider capability for
each module's data. Read and control are separate grants, so a clickable module
that would acquire control has to degrade to a plain readout when only read is
granted — omit the binding rather than bind a command that will be denied.

**Caller evidence.** `bar/lua/bar/init.lua`: the group plan and anchors at
`:139`-`:145`, the default module plan at `:147`-`:153`, the single-space join
and its rationale at `:155`-`:159`, the module renderers (clock at `:2168`,
network at `:2808`, and the presence-conditional providers around `:2975`-
`:3810` — bluetooth, backlight, idle, microphone), the strip assembly in
`render_bar` at `:4453`-`:4479`, and the empty-band sentinel at `:529`.

**Accessibility notes.** A status strip is a text row: its accessible content is
the concatenation of visible atoms, so every atom has to read sensibly on its
own and in order (`cpu 12%`, not `12%`). Order carries meaning, so do not
re-order modules by availability — hide and keep the rest in declared order.
Keyboard reachability is absent by construction; any control a module exposes
needs a command entry outside the band.

**Bounds.** One block for the strip and one atom per module plus one per capsule.
The strip is the surface most likely to grow with data, so a pattern fixes its
module list at activation and re-emits the whole row on change instead of
appending.

### Example: status strip

```lua
-- Manifest grants ui.rich and platform.cpu-read; no control grants.
local GROUP_ORDER = { { key = "left", anchor = "left" }, { key = "center", anchor = "center" }, { key = "right", anchor = "right" } }

local function module(name, text, interactive)
  local node = { kind = "Text", text = text }
  if interactive then
    node.on_click = {
      command = "example-plugin:detail",
      args = { module = name },
      region = { id = "module:" .. name, kind = "module" },
    }
  end
  return node
end

local function strip(state)
  local plan = {
    left = { module("workspaces", " 1*   2 ") },
    center = { module("clock", state.clock) },
    right = {},
  }
  -- Omit, never pad: an ungranted provider contributes no atom at all.
  if state.cpu ~= nil then
    plan.right[#plan.right + 1] = module("cpu", "cpu " .. state.cpu, true)
  end
  if state.memory ~= nil then
    plan.right[#plan.right + 1] = module("memory", "mem " .. state.memory, false)
  end

  local children = {}
  for _, group in ipairs(GROUP_ORDER) do
    local atoms = plan[group.key]
    if #atoms > 0 then
      local joined = {}
      for index, atom in ipairs(atoms) do
        if index > 1 then
          joined[#joined + 1] = { kind = "Text", text = " " }
        end
        joined[#joined + 1] = atom
      end
      children[#children + 1] = { kind = "Row", anchor = group.anchor, children = joined }
    end
  end
  return { kind = "Row", children = children }
end

local handle = bitty.ui.mount("bottom", strip({ clock = "12:04", cpu = "12%" }))
```

## Width hooks: what the landed table does not unlock

`min_width` and `max_width` are the fifth and sixth of the ten style keys, and
`gap` — the sibling spacing, an integer in cells capped at `32`, accepted inside
a `Row`/`Column`/`List` and refused on a `Text` leaf — is the ninth, behind the
`min_height` / `max_height` pair. It is still tempting to read the width pair as
the missing lever for equal-width capsules, a bounded tooltip, a fixed-width
sheet, or a stable module column. They are not, on any of the five surfaces
these entries use, and the reason is one line of scope selection rather than a
gap in the primitives. `gap` changes nothing about that:
it separates siblings, it never sizes a node, so it is not a width substitute on
any surface — and it is rich-reader-only, like the width pair.

| Entry | What `min_width` / `max_width` would do              | Reader that accepts it                 | Shipped caller using it | Verdict                                                                     |
| ----- | ---------------------------------------------------- | -------------------------------------- | ----------------------- | --------------------------------------------------------------------------- |
| E-1   | equalise or cap an anchored group                    | rich reader only; the band drops it    | none                    | **gap** on the band surface                                                 |
| E-2   | give capsules one width, or fix a pill track's width | rich reader only; the band drops it    | none                    | **gap**: the rich reader rejects the `bg` fill a capsule needs              |
| E-3   | bound a tooltip body                                 | rich reader only; the overlay drops it | none                    | **gap** for the shipped overlay rung; rich-path text shaping **unverified** |
| E-4   | fix a sheet's width                                  | rich reader only; the overlay drops it | none                    | **gap**: the sheet surface is the overlay, which drops `style`              |
| E-5   | bound a module readout                               | rich reader only; the band drops it    | none                    | **gap** on the band surface                                                 |

Three conclusions, all negative, all worth recording rather than working around:

1. **The width hooks do not close the pill gap.** E-2 needs a fill, `bg` belongs
   to the band reader, and the width keys belong to the rich reader. A pill that
   is both filled and width-equalised would need the band path to paint `style`
   or the rich reader to accept a fill token; neither exists, so the missing
   primitive is unchanged.
2. **No shipped caller declares the table.** Across the published plugin
   collection at `bitty-plugins@4197ff9`, no Lua source under `plugins/*/lua`
   carries a `style` key, a `min_width`, or a `max_width` — and `on_hover`, the
   one binding that would drive a rich-path tooltip, has a single caller, in
   `bar`. Every rich-path statement in this document is therefore bridge-verified
   and **caller-not-demonstrated**, and the entries say so rather than implying a
   shipped consumer.
3. **Width is a measure bound, and its text behaviour is unpinned here.** Core
   clamps the content box (`SceneNodeStyle::constrain_width`,
   `bitty-rich/src/scene.rs:619`), and a validated range is a two-sided clamp,
   never a precedence rule. Whether a `Text` child of a narrowed box wraps,
   clips, or overflows is not stated by any artifact this catalog cites, so any
   entry that would depend on it is **unverified**, not shipped.

A fourth fact belongs here because it is the one an author will trip over: the
SDK reference page that owns the slot and budget tables is pinned at
`bitty@94039dd1`, but it records the style table from `bitty` main — `4bc0fce5`
for the table, `0a7f1f15` for the height rows — and that table is the same
ten-key set the bridge at `origin/main` (`0a7f1f15`) parses. The page's other
pins still describe its own revision, and this catalog cites the bridge by
`file:line` so a reader can tell which revision a claim comes from.

## Security review

These patterns change nothing about the security posture: each is a composition
of already-admitted pieces. The properties that still hold are enforced by the
host.

| Concern                | Position                                                                                                                                                         |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Presentation not truth | Every pattern mounts a declarative subtree; none writes grid, cursor, mode, scrollback, or layout.                                                               |
| No hot-path execution  | Bindings dispatch command ids; no plugin VM runs on the input path, and no pattern polls the host per frame.                                                     |
| Bounded inputs         | Node, depth, text, block, and retained-string budgets apply per generation; a non-flat `style` costs one node and one depth level and is refused at the ceiling. |
| Authority separation   | Draw authority (`ui.present`) never implies input authority (`ui.overlay.focus`); no pattern asks for both to fake a tooltip.                                    |
| No ambient geometry    | Region ids derive rectangles from painted spans and carry no coordinates; no pattern reads or sets absolute positions.                                           |
| Fail closed            | A denied grant, declined notification, rejected mount, or over-cap style leaves the rest of the surface unchanged.                                               |
| No modal authority     | No pattern paints a backdrop, claims modality, or traps focus; the single modal authority stays Core-owned.                                                      |

No capability, budget, or trust decision is proposed here, so no `P0` criterion
is affected. A security reviewer is required if a future revision admits a new
node kind, an anchored surface, an input-bearing band, or a plugin modal.

## Verification plan

A revision of this catalog that moved to `accepted` would need at least:

1. `just check` green in this repository with zero markdownlint, link, metadata,
   language, agents, and hygiene issues.
2. A mount test for each of E-1, E-2, E-3 (rungs 1-3), and E-5 using only the
   fields the entry lists, with no additional field accepted.
3. A rejection test for each excluded shape: `Image`, `Table`, `Rule`, a
   `style` key on a band node that paints nothing, an `anchor` on the root or
   below the root's direct children, and `fg`/`bg` on a rich node.
4. A dispatch test proving E-2, E-3, and E-5 bindings route from `top`/`bottom`,
   that `on_hover` requires a region and injects `phase`, and that nothing routes
   from an overlay block.
5. An omission test proving a missing module or group emits no atom and no
   placeholder while the remaining atoms are unchanged.
6. A fallback test proving E-3 degrades in order: inline text, then the hover
   tooltip, then a notification, and that a declined notification counts as
   "not shown".
7. A negative test proving E-4 cannot be built: no backdrop, no modality claim,
   no focus trap, and `Esc` on the presentation overlay dismissed without
   consuming the press.
8. A budget test proving a strip or pill row that exceeds the node budget fails
   closed rather than truncating silently, and that a styled tree at the depth
   ceiling is refused.
9. The caller-evidence paths in this document still resolve at the next revision;
   a pattern whose caller was deleted or renamed is retired rather than left
   stale.
10. A width-hook test per entry surface: a rich `terminal` mount that declares
    `min_width` / `max_width` inside the caps and asserts one extra node and one
    extra depth level, plus a rejection test proving an inverted range fails
    instead of being reordered.
11. A drop test proving the same declaration on a band and on the overlay mounts,
    reports once per generation through `ui_style_ignored`, and paints nothing.
12. A vocabulary test proving no example in this document uses a kind outside
    `Text | Row | Column | List` and no slot outside the seven — with
    `statusline` rejected by name.
13. A shaping test for the **unverified** item in E-3: a `Text` child of a
    `max_width`-bounded box, asserting whether it wraps, clips, or overflows
    before any entry describes tooltip wrapping as available.

## Alternatives considered

| Alternative                                          | Trade-off                                                                              | Disposition                                                                          |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Add real component constructors (`Pill`, `Sheet`, …) | Familiar authoring surface, but changes the accepted node set.                         | Rejected; the v1 surface composes from four kinds, and additions need a new surface. |
| Ship the patterns as a shared Lua library            | Removes duplication, but creates a versioned dependency with no owner yet.             | Deferred; the `bitty-ui-core` direction is candidate and unowned.                    |
| Keep the earlier catalog and add notes               | Smallest diff, but leaves two documents disagreeing about the same fields.             | Rejected; one authoritative catalog, the older record becomes a pointer.             |
| Document the bar's source shape only                 | Cheap, but ties every plugin to one implementation's choices.                          | Rejected; the entries state the shape and let tokens and caps differ.                |
| Treat the presentation overlay as a modal sheet      | Reuses an existing host, but transfers modal authority to plugins.                     | Rejected; the surface is non-modal by contract and paints only.                      |
| Treat a styled rich node as the pill primitive       | Gets radius and border immediately, but loses the token fill that makes a pill a pill. | Rejected as a recipe; recorded as the fill-token gap instead.                        |

## Affected contracts

| Contract                                                                                                                                                                | Effect                                                                                                                                                                                                                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted)                                                                                     | Unchanged; the catalog composes only its accepted fields and gates.                                                                                                                                                                                                                                       |
| [bitty.ui reference](../sdk/reference/ui.md) (draft)                                                                                                                    | Supplies the band token table, binding shape, dispatch scope, and budgets; its own hover, anchor, and `style` coverage is a residual below.                                                                                                                                                               |
| [Plugin UI Slot Inventory](plugin-ui-slot-inventory-candidate.md) (candidate)                                                                                           | Supplies per-slot multiplicity and conflict rules the entries assume.                                                                                                                                                                                                                                     |
| [Lua UI Component Model](lua-ui-component-model-candidate.md) (candidate)                                                                                               | Records the component library and accessibility direction these patterns precede.                                                                                                                                                                                                                         |
| [Scene Style Primitives](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md) (candidate, shipped slice) | Supplies the `style` caps and token set the entries quote. The candidate text closes the _paint_ set at four primitives while the bridge parses ten (four paint keys plus `min_width`, `max_width`, `min_height`, `max_height`, `gap`, and `align`), and the band path is outside both — see residual 11. |
| [bitty.ui Surface Bounds and Capability Boundary](../sdk/reference/ui-surface-bounds.md) (draft, pinned at `bitty@94039dd1`)                                            | Supplies the closed slot, node, and budget tables this record links instead of restating; its "Landed style table" section records the same ten-key table this record cites from the bridge.                                                                                                              |
| [Plugin UI Component Catalog](ui-component-catalog-candidate.md) (deprecated)                                                                                           | Superseded by this record; retained as a pointer.                                                                                                                                                                                                                                                         |
| [Cross-Plugin Composition](cross-plugin-composition-candidate.md) (candidate)                                                                                           | Would own any future shared pattern package or service.                                                                                                                                                                                                                                                   |
| Theme Token Contract (candidate, terminal-side)                                                                                                                         | Owns token naming and the fail-closed disposition of unknown tokens.                                                                                                                                                                                                                                      |

## Open points

### Residuals

1. **The bar migration is not done, and this record does not do it.** Checkbox 3
   of the tracking issue — "bar migrated onto catalog entries (dogfood proof)" —
   is code in the plugin repository. The bar still hand-rolls every pattern it is
   cited for here; the `file:line` evidence is evidence of a hand-rolled caller,
   not of a migration. Until that lands, this catalog is a description of shipped
   compositions, and there is no shared implementation to point at.
2. **No anchored tooltip or popover primitive.** Rung 4 of E-3 cannot be built: no
   anchoring, offset, placement, or per-item geometry exists, and an overlay block
   is presented over the focused view rather than beside a band atom. Region ids
   give a stable identity, not a rectangle a plugin can position against.
3. **No modality.** E-4 stays a gap: no backdrop, focus trap, stacking tier,
   dismissal ownership, or announcement channel. The substitutes are non-modal by
   contract.
4. **No style on the band or overlay path, and no width there either.** The
   `style` table is parsed for the rich vocabulary only; on a band or overlay node
   it is an unknown optional key that the bridge tolerates, that the paint path
   ignores, and that the mount path reports once per generation so the author
   learns it painted nothing. That includes the width, height, and spacing keys:
   `min_width`, `max_width`, `min_height`, `max_height`, and `gap` are as inert
   on a band as `radius` is. A rounded, padded, width-set, or sibling-spaced
   band, capsule, tooltip, or sheet therefore needs the band path to consume the
   style table — a separate decision with its own budget and paint consequences.
5. **No fill token in the style table.** `border_token` colours a stroke; a styled
   node has no way to name a fill, so radius, width, and token fill cannot be
   combined today. That is still the single missing primitive between "square
   padded capsule" and a real pill, and the width pair and `gap` do not change
   it: they let a rich node hold a width, not wear a fill.
6. **No interaction-state styling.** There is no hover, focus, or selected state
   field, so highlighting a pill under the pointer requires a re-mount. A
   restyle-on-state pattern would need a state vocabulary the surface does not
   have.
7. **The `bitty.ui` reference page lags the build.** Its pinned revision does not
   yet document `on_hover`, the `anchor` group field, or the `style` table. Those
   three are now recorded with `bitty` `file:line` evidence in
   [bitty.ui Surface Bounds and Capability Boundary](../sdk/reference/ui-surface-bounds.md);
   reconciling the reference page's own placement table is a remaining
   documentation task in this repository, not a contract change.
8. **Per-slot aggregate bounds across plugins**, which would bound a composing
   band's total size, are still an open inventory question.
9. **Ownership of a shared implementation**, if these patterns ever become a
   versioned library, remains unresolved (`bitty-ui-core` direction, candidate and
   unowned).
10. **No shipped caller declares the style table or any of its ten keys.** No
    Lua source under `plugins/*/lua` in the published plugin collection
    (`bitty-plugins@4197ff9`) carries `style`, `min_width`, `max_width`,
    `min_height`, `max_height`, or `gap`, and `on_hover` has exactly one caller,
    in `bar`. Every rich-path hook
    described here is therefore bridge-verified and
    **caller-not-demonstrated**: the primitives exist, and nothing shipped wears
    them. A caller would be the first evidence that the shape works end to end,
    and no entry claims one.
11. **The style table is landed code ahead of the contract text.** The candidate
    record closes the paint set at four primitives
    (`specifications/scene-style-primitives-candidate.md:145`); the bridge parses
    the additive six — `min_width`, `max_width`, `min_height`, `max_height`,
    `gap`, and `align` — and the SDK page that owns the slot and budget tables
    records the same ten-key table
    ([bitty.ui Surface Bounds and Capability Boundary](../sdk/reference/ui-surface-bounds.md),
    "Landed style table"). The bridge at `origin/main` (`0a7f1f15`) parses ten
    (`UI_STYLE_FIELDS: [&str; 10]`, `ui_style.rs:279-290`), the last of them
    `align`. Reconciling the candidate record with the ten-key bridge is a
    documentation task in that repository, not a capability this catalog may
    claim on its behalf.
12. **Narrowed-box text shaping is unverified.** `constrain_width` sizes a
    content box (`bitty-rich/src/scene.rs:619`) and a validated range is a
    two-sided clamp. Whether a `Text` child of a `max_width`-bounded box wraps,
    clips, or overflows is pinned by no artifact this catalog can cite, so E-3
    marks the width behaviour of a tooltip body **unverified** rather than
    describing it as available.
13. **`left` and `right` are column bands, so two entries cannot compose there.**
    Both slots require a column-shaped root and validate as `Unanchored`
    (`band_slots.rs:36-38`, `:241-259`; `services.rs:6749-6752`), which means the
    `Row`-rooted entries E-1 and E-5 mount into `top`/`bottom` only. A vertical
    variant is a different composition, and this catalog does not describe one.

### Resolved by the landed build

- **The band path does not consume the `style` table.** The rich vocabulary hosts
  `style = { padding, radius, border, border_token, min_width, max_width, min_height, max_height, gap, align }`;
  on a band or overlay node the same key is an unknown optional field that
  mounts, paints nothing, and is reported once per generation so the author
  learns the declaration did not take effect. The key set, caps, the
  one-stroke-per-node rule, and that loud-drop path are recorded in
  [bitty.ui Surface Bounds and Capability Boundary](../sdk/reference/ui-surface-bounds.md)
  as the same ten-key table this catalog cites from the bridge.
  Whether the band path should one day _paint_ the table stays an
  accepted-extension decision owned by the terminal-side contract, so this record
  no longer puts the question as if its answer were unknown.
- **Width is a declared range, not plugin layout.** `min_width` / `max_width`
  bound the content box in cells, `0` means unbounded on that side, an inverted
  range is refused, and Core clamps in the measure path
  (`ui_style.rs:279-290`, `:879`; `bitty-rich/src/scene.rs:619`). The
  question "can a plugin size its own node?" is answered for the rich reader; it
  is a plain no for bands and the overlay, where the key never reaches a paint.
- **Height is the same declared range on the other axis.** `min_height` /
  `max_height` bound the same content box, `0` means unbounded on that side, an
  inverted range is refused, and Core applies the pair in the measure and paint
  paths (`ui_style.rs:279-290`, `:902`; `bitty-rich/src/scene.rs:640`). No kind
  refuses it, because a height is the node's own extent rather than a relation
  between siblings.
- **Sibling spacing is a declared gap, not plugin layout.** `gap` spaces adjacent
  siblings in cells, inclusive `0..=32`, `0` renders identically to absent, and a
  non-zero value on a kind that composes no siblings is refused rather than
  dropped (`ui_style.rs:279-290`, `:340`; `ui_rich.rs:460`). It answers
  "can a plugin space its children?" for the rich reader, and it is a plain no
  for bands and the overlay, which drop the table.

### Decisions still required

- Whether the style table gains a fill token, and whether that token belongs to the
  theme content style rather than the node declaration.
- Whether the band or overlay reader ever consumes `style` at all, and if so
  whether it takes the width and spacing keys with the rest of the table — a
  decision with its own budget and paint consequences, since it would turn every
  `Row`-group entry into a width-setting and spacing-setting surface.
- Whether a narrowed rich content box wraps, clips, or overflows, and which of the
  three the catalog should describe for a bounded tooltip or sheet body.
- Whether an anchored popup surface becomes a primitive, and what anchoring
  contract it would need to avoid leaking geometry to plugins.
- Whether a plugin modal surface ever exists, and how it would relate to the single
  Core-owned modal authority.
- The accessibility property schema (role, name override, state) and whether band
  atoms can carry any of it.
- Whether the bar repository migrates onto these entries, and what evidence would
  count as dogfood proof.

## Acceptance criteria

1. Every entry uses only fields that exist in the current build, or is marked
   **partial** or **gap** with the missing primitive named.
2. No entry implies a new node kind, slot, capability, budget, or error code.
3. Each entry names a shipped caller at `file:line` and states whether the pattern
   is expressible today.
4. Each entry carries a contract block, a scene shape, style hooks, slot and
   capability, accessibility notes, bounds, a headed example, and a proof status.
5. Every entry names the slot(s) it uses from the closed seven, and the retired
   `statusline` name appears in no entry as a slot.
6. Every entry's composition uses only `Text`, `Row`, `Column`, `List`; the five
   v1-excluded kinds appear in no scene shape and in no example.
7. Every entry states whether the style and width hooks reach its surface, marks a
   hook the surface drops as a **gap**, and marks behaviour no artifact proves as
   **unverified** rather than describing it as shipped.
8. No entry, example, or summary is described as headed verification, and no
   screenshot or run log is claimed.
9. The residuals section states plainly that checkbox 3 (bar migration) remains
   open and is code, not documentation.
10. Review by a plugin-ecosystem category owner and a docs curator, with
    terminal-side coordination for any entry that touches host geometry or the
    overlay contract.
11. Acceptance happens through the owning contract (the UI component or slot
    inventory track), not by flipping this record's status.

## P0 Review Sign-off

Not applicable: no security boundary, capability, resource ceiling, or trust
decision changes; every pattern is a composition of already-admitted pieces and
narrows authoring rather than widening authority.

## Revision history

| Revision      | Change                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Initial       | Five entries (E-1 to E-5) with scene shape, style hooks, slot and capability, caller evidence at `file:line`, accessibility notes, bounds, and one headed example each; the residual list, including the open bar-migration deliverable.                                                                                                                                                                                                                                                                                                                    |
| This revision | Adds a per-entry contract block (closed-set slot, v1 node composition, style, width and spacing hooks, interaction route, budget footprint, proof status); a composition register; the `min_width` / `max_width` width hooks, the `min_height` / `max_height` height pair, the `gap` sibling-spacing key, and `align` — the ten-key style table — with an explicit statement that they reach no entry's surface; and residuals 10-13 covering caller absence, the four-primitive contract text, unverified narrowed-box shaping, and column-band placement. |

## References

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) —
  accepted slot set, node subset, handle model, and capability gates.
- [bitty.ui reference](../sdk/reference/ui.md) — band token table, binding shape,
  band-only dispatch, overlay style parity, overlay capture, presentation-only
  overlay, and per-generation bounds.
- [bitty.ui Surface Bounds and Capability Boundary](../sdk/reference/ui-surface-bounds.md)
  — closed slot, node, and style vocabularies, capability map, contract budgets,
  one-stroke rule, style loud drop, and the not-in-v1 list.
- [Scene Style Primitives (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/scene-style-primitives-candidate.md)
  — the four-primitive style table, its caps, and its resolution order.
- [Plugin UI Slot Inventory (Candidate)](plugin-ui-slot-inventory-candidate.md) —
  per-slot purpose, multiplicity, bounds, and conflict resolution.
- [Lua UI Component Model (Candidate)](lua-ui-component-model-candidate.md) —
  five-level component hierarchy, composition principle, and accessibility
  direction.
- [Cross-Plugin Composition (Candidate)](cross-plugin-composition-candidate.md) —
  host-mediated reuse rules for any future shared pattern package.
- [Plugin UI Component Catalog (Deprecated)](ui-component-catalog-candidate.md) —
  the superseded earlier revision.
- [Plugin system](plugin-system.md) — presentation-not-truth boundary and
  extension levels.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  — `SceneNode`, `RichBlock` replacement, and scene bounds.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  — panel identity, focus routing, and Core-owned modal authority.
- [Theme Token Contract (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/theme-token-contract-candidate.md)
  — token naming and unknown-token disposition.
- [bitty.notify reference](../sdk/reference/notify.md) — the notification rung's
  payload, gate, and acceptance semantics.
- [Bar plugin design](../docs/plugins/bar/design.md) and
  [bar schemas and contracts](../docs/plugins/bar/schemas.md) — the reference
  implementation's pinned presentation and scene contract.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) — resource
  ceilings no pattern may widen.
