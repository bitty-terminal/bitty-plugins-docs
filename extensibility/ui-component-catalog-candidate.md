---
title: Plugin UI Component Catalog (Candidate)
description: Candidate catalog of reusable plugin UI composition patterns over the accepted v1 scene nodes with scene shapes style hooks accessibility notes and headed examples
category: extensibility
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 30
---

# Plugin UI Component Catalog (Candidate)

> Status: **draft candidate** — not **Accepted**, not **Verified**, not
> **Compatible**, and not normative. This document catalogues reusable
> composition recipes over the accepted Plugin API v1 declarative surface. A
> recipe is documentation of how accepted pieces compose; it is not an API. No
> entry here adds a node kind, slot, style hook, interaction binding,
> capability, budget, or error code, and no entry relaxes the accepted
> [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md).
> Where a pattern needs something the accepted surface does not have, the entry
> says so and is marked **Not available** rather than sketched as if it worked.

## Purpose and scope

Downstream extensions keep rebuilding the same surfaces by hand: an edge band, a
row of status pills, a detail hint, a transient sheet, a status strip. Each
plugin re-derives the scene shape, the style tokens, and the interaction
bindings, and each gets slightly different answers. The bar plugin is the
reference implementation for the useful cases, but its shape is discoverable
only by reading its source.

This catalog turns that into a documented contract-shaped record: five named
composition patterns, each with a scene shape, its style hooks, its interaction
and accessibility notes, its budget behaviour, and a headed example. The goal
recorded by the request is that a plugin author composes from documented recipes
instead of hand-rolling `SceneNode` trees.

In scope:

- C-1 edge band, C-2 pill row, C-3 tooltip and detail fallback, C-4 modal sheet,
  C-5 status strip.
- For every entry: scene shape, style hooks, interaction, accessibility notes,
  budget note, and a headed Lua example.
- The bar plugin mapping: which bar element corresponds to which entry, as
  evidence that the recipes describe real compositions rather than invented
  ones.
- A per-entry availability verdict: **Accepted v1**, **Partially available**, or
  **Not available**.

Out of scope and owned elsewhere:

- the closed slot set, node subset, handle model, style fields, and interaction
  bindings (accepted,
  [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md);
  current host facts in the [bitty.ui reference](../sdk/reference/ui.md));
- per-slot purpose, multiplicity, and conflict resolution (candidate,
  [Plugin UI Slot Inventory](plugin-ui-slot-inventory-candidate.md));
- the five-level component hierarchy and the `bitty-ui-core` library direction
  (candidate, [Lua UI Component Model](lua-ui-component-model-candidate.md));
- host layout, band geometry, decoration, and paint order (terminal-side; the
  [bitty.ui reference](../sdk/reference/ui.md) states the host facts);
- panel identity, activity stacks, and the focusable overlay contract (accepted,
  [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md));
  `register_panel` and `PanelProvider` remain excluded from plugin API v1;
- token names and theme resolution (the
  [Theme Token Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/theme-token-contract-candidate.md),
  candidate);
- cross-plugin sharing of a recipe implementation (candidate,
  [Cross-Plugin Composition](cross-plugin-composition-candidate.md)).

## Normative sources this specification must not weaken

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (accepted): the seven-name closed slot set, the v1 node subset (`Text`,
  `Row`, `Column`, `List`), the opaque generation-owned `block_id`, the
  `ui.rich` / `ui.overlay` gates, and the exclusion of `register_panel`.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  (accepted): the `RichBlock` replacement rule and scene bounds `SCN-1`..`SCN-5`
  (2048 nodes, depth 32, 256 KiB text per block, 2 MiB aggregated rich bytes,
  64 blocks per terminal).
- [bitty.ui reference](../sdk/reference/ui.md) (draft; current host fact): the
  style fields (`fg`, `bg`, `bold`), the closed theme-token table, the
  `on_click` / `on_scroll` command bindings with `region` identities, band-only
  interaction routing, overlay capture, presentation-only overlay, and the
  per-generation bounds actually enforced.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  (accepted): panel identity, focus routing, and the single Core-owned modal
  authority.
- [Plugin system](plugin-system.md) (draft): plugins alter presentation, never
  Terminal Truth; capabilities are deny-by-default; composition goes through
  declared host surfaces.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (accepted):
  per-plugin resource ceilings that no recipe may widen.
- [Security Overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md)
  and [Threat Model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md):
  presentation-not-truth, no hot-path execution, bounded inputs.

If any recipe here contradicts a normative source, the normative text wins. The
catalog has no authority to promote a candidate into a contract.

## Terminology and entry status

| Term            | Meaning in this document                                                                                                    |
| --------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Recipe          | A named composition of accepted scene nodes, style fields, and bindings. Documentation, not API.                            |
| Scene shape     | The node tree a recipe mounts, expressed with the four v1 kinds.                                                            |
| Style hook      | A field the host paints: `fg`, `bg` (theme-token names), or `bold`. No other appearance field exists in v1.                 |
| Token           | A host-resolved name from the closed table `foreground`, `background`, `cursor`, `selection`, `accent`. Never a raw colour. |
| Group           | A `Row` or `Column` child of the root used for anchoring; groups carry no geometry of their own.                            |
| Atom            | One `Text` leaf, the only unit that carries visible characters in v1.                                                       |
| Recipe status   | Always **candidate** in this document; no recipe is a contract.                                                             |
| Surface status  | Whether the fields a recipe uses are **Accepted v1**, **Partially available**, or **Not available** today.                  |
| Detail fallback | The secondary presentation a surface uses when its primary presentation cannot show the whole detail.                       |

Every entry carries the same six fields: scene shape, style hooks, interaction,
accessibility notes, budget note, and a headed example. Availability is stated
before the fields, because a recipe that needs a missing field is not a recipe.

| ID  | Pattern                   | Where it mounts               | Nodes and fields used                         | Surface status                                                                        |
| --- | ------------------------- | ----------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------- |
| C-1 | Edge band                 | `top`, `bottom`               | `Row`, `Text`; `fg`, `bg`, `bold`             | **Accepted v1**                                                                       |
| C-2 | Pill row                  | inside C-1 or C-5             | `Row`, `Text`; `fg`, `bg`, `bold`, `on_click` | **Accepted v1**; no radius, border, or padding field                                  |
| C-3 | Tooltip / detail fallback | `bottom`, `overlay`, `notify` | `Row`, `Column`, `Text`; `fg`, `bg`, `bold`   | **Partially available**: band and notification rungs accepted; overlay rung host fact |
| C-4 | Modal sheet               | `overlay`                     | none that grants modality                     | **Not available** in plugin API v1                                                    |
| C-5 | Status strip              | `bottom`                      | `Row`, `Text`; `fg`, `bg`, `bold`             | **Accepted v1**                                                                       |

## Rules that apply to every recipe

1. **Compose only from `Text`, `Row`, `Column`, and `List`.** `Block`, `Image`,
   `CodeBlock`, `Table`, and `Rule` are rejected before any host call with
   `E_UI_COMPONENT_INVALID`. A recipe that would need one of them is marked
   **Not available**, not approximated with a lying shape.
2. **One root, one block, one handle.** A surface mounts one root node and is
   updated through its `block_id`. Updating replaces the subtree; it never
   accumulates siblings.
3. **Text is the only atom.** Labels, separators, padding, marks, and counts are
   all characters inside `Text` leaves. There is no spacer, divider, icon, or
   image node.
4. **Style is tokens plus bold.** `fg` and `bg` name tokens from the closed
   table; an unknown well-formed token falls back to the span default and is
   counted in host diagnostics. Raw colour strings are rejected. `bold` paints
   the host's synthetic double strike.
5. **Spacing is literal.** Padding is one or more spaces inside the atom that
   owns the fill, so the fill and its padding are one region and one hit target.
   There is no padding, margin, gap, alignment, width, or radius field in v1.
6. **Interaction is a command binding, never a callback.** `on_click` and
   `on_scroll` carry `{ command, args }`, optionally with a per-item
   `region = { id, kind = "item" | "module" }`. Command ids are at most 64
   bytes, string argument values at most 256 bytes, region ids at most 64 bytes,
   and the dispatched argument table is capped at 16 entries before
   host-injected entries. No plugin VM runs on the input path.
7. **Pointer routing is band-only.** Bindings dispatch only from `top` and
   `bottom`. An `overlay` block's bindings are retained and not routed, because
   no overlay hit-test contract exists.
8. **Omit, do not placeholder.** A group whose content is unavailable is omitted
   from the tree. A recipe never paints an empty capsule, a dash, or a zero as a
   stand-in for data it does not have.
9. **Emit on state change only.** Lua re-emits on a state or service event; it
   never runs a per-frame draw loop.
10. **Budgets are per generation and fail closed.** 2048 nodes, 256 KiB text,
    and a bridge depth bound of 16 per block; 64 retained blocks and 2 MiB
    retained string bytes per generation. Exceeding a scene bound fails with
    `E_UI_COMPONENT_INVALID`; exceeding a registry bound fails with
    `E_UI_BLOCK_BUDGET`.

## C-1 Edge band

**Surface status: Accepted v1.** Mounts into `top` or `bottom` under `ui.rich`.

**Scene shape.** A single `Row` root with up to three anchored groups, each a
`Row` of atoms, emitted only when non-empty:

```text
Row
├── Row   (left group,  anchored to the leading inset)
├── Row   (center group, centered across the band)
└── Row   (right group, aligned to the trailing inset)
```

The band's thickness, insets, stacking against other plugins' bands, and paint
order are host-owned. Two plugins on one edge stack in plugin id byte order, so
a recipe never encodes a position.

**Style hooks.** `fg`, `bg`, and `bold` on atoms and on group containers; a
container's declaration reaches its rows, so the root is the broadest lever and
an atom's own declaration wins. There is no band-level background, border,
height, alignment, or width field.

**Interaction.** `on_click` and `on_scroll` on atoms, optionally with a
`region` identity. Declaring different regions on the two bindings rejects the
mount. Chrome clicks and scrolls never move focus, never change layout, and
never start a selection.

**Accessibility notes.** A band receives no keyboard or IME input, so every band
item is pointer-reachable only; the recipe therefore requires a command or
keymap path for the same action, and the command id is what a help surface
documents. The visible text is the only accessible name the v1 surface can
carry: there is no `role`, `label`, `description`, or state field, and the
accessibility property schema remains candidate and owner-pending (U-9 in
[Lua UI Component Model](lua-ui-component-model-candidate.md)). Do not claim
that a band contributes to an accessibility tree.

**Budget note.** One retained block per band surface. The host owns band
thickness and available width and does not report either to the plugin, so the
recipe bounds every label with an explicit character budget instead of assuming
space it cannot measure.

### Example: edge band

```lua
-- Manifest grants ui.rich.
local function band(cpu)
  return {
    kind = "Row",
    children = {
      {
        kind = "Row",
        children = {
          { kind = "Text", text = " 1 ", fg = "background", bg = "accent", bold = true },
          { kind = "Text", text = " " },
          { kind = "Text", text = " 2 ", bg = "selection" },
        },
      },
      { kind = "Row", children = { { kind = "Text", text = "12:04" } } },
      { kind = "Row", children = { { kind = "Text", text = "cpu " .. cpu } } },
    },
  }
end

local handle = bitty.ui.mount("bottom", band("12%"))
bitty.ui.update(handle, band("9%")) -- true: subtree replaced, handle unchanged
```

## C-2 Pill row

**Surface status: Accepted v1.** Composes inside C-1 or C-5; the row itself is a
`Row` of capsule atoms.

**Scene shape.** Each pill is one `Text` atom whose visible string is
`" " .. label .. marks .. " "`, where the padding shares the atom's `bg` so the
whole capsule is one run and one click target. Marks append after the label in a
fixed order: for example `*` active, `+` occupied, then `!` attention, in every
label mode.

```text
Row
├── Text  " 1* "   fg=background bg=accent bold=true   on_click={...}
├── Text  " 2 "    bg=selection                        on_click={...}
└── Text  " 3+! "  bg=selection                        on_click={...}
```

There is no `Capsule`, `Button`, `Badge`, or `Tab` node. A capsule is a styled
`Text` atom; a separate node kind would be a v2+ surface change.

**Style hooks.** Fill via `bg`, label via `fg`, emphasis via `bold`. The
reference implementation uses `background` on `accent` for the active capsule
and `selection` for inactive capsules, with `cursor` for attention emphasis and
attention winning over occupied emphasis. Those are token choices available to
any recipe, not reserved names. Because v1 exposes no corner radius, capsules
are square padded fills; a rounded pill needs a radius primitive that does not
exist.

**Interaction.** One `on_click` binding per pill carrying the command and a
bounded `args` table, optionally with `region = { id = "<stable-id>", kind = "item" }`.
Region ids derive the painted span rectangle; they carry no coordinates, and a
region-aware binding reserves `region` and `button`, so at most 14 declared
arguments remain. Keep arguments declarative identity, not state snapshots.

**Accessibility notes.** Each pill's accessible text is its visible label plus
marks; if marks encode state, the recipe must also expose that state in a
command-visible or notification-visible place, because no state property exists.
Pointer-only reachability means the same action needs a keymap or command
entry. Do not encode selection in colour alone: the active capsule in the
reference implementation is both filled differently and bold, which survives a
monochrome theme.

**Budget note.** One atom per pill; no per-pill block. A row of tags that grows
without bound is a node-budget failure, so the recipe caps the visible count and
carries the remainder in a single overflow atom.

### Example: pill row

```lua
-- Manifest grants ui.rich and declares the focus command.
local row = { kind = "Row", children = {} }

local function pill(label, active, marks)
  return {
    kind = "Text",
    text = " " .. label .. (marks or "") .. " ",
    fg = active and "background" or nil,
    bg = active and "accent" or "selection",
    bold = active or nil,
    on_click = {
      command = "example-plugin:focus",
      args = { target = label },
      region = { id = "pill:" .. label, kind = "item" },
    },
  }
end

row.children[#row.children + 1] = pill("1", true, "*")
row.children[#row.children + 1] = pill("2", false)

local handle = bitty.ui.mount("bottom", row)
```

## C-3 Tooltip and detail fallback

**Surface status: Partially available.** There is no tooltip, popover, hover, or
focus-in-band surface in v1. The recipe is a fallback ladder, and only the first
two rungs are contracted.

**Scene shape.** Four rungs, in preference order:

```text
1. inline      Text suffix inside the same band atom          (Accepted v1)
2. notify      bitty.notify.show { title, body?, urgency? }   (Accepted v1, platform.notify)
3. present     overlay block drawn by bitty.ui.present.show   (host fact, not in the accepted RFC)
4. anchored    a detail surface positioned near its trigger   (Not available)
```

Rung 4 is what authors actually mean by "tooltip", and it cannot be built: no
anchoring, offset, or per-item geometry exists, and an `overlay` block is
centered over the focused view, not placed against a band atom. Rung 3 is a
read-only card above the focused view, never a hover bubble.

**Style hooks.** Rung 1 uses the band's own style fields. Rung 3 uses the same
closed token table and the same bold double strike as bands, because band and
overlay paint through one style path; an unstyled overlay row keeps the single
default pill. Rung 2 has no style hooks at all: notification rendering, rate
policy, and delivery are host-side, and acceptance is not delivery.

**Interaction.** There is no hover event, no pointer-enter binding, and no
per-item routing on overlay surfaces. Detail must be asked for: a `detail`
command bound to `on_click` on the band atom, or a keymap entry, or a cold-path
poll of `bitty.ui.overlay.poll`. A recipe never assumes a tooltip appears on
hover.

**Accessibility notes.** Because there is no hover, no focus target, and no
`description` field, detail text is not attached to its trigger for assistive
technology. A detail command that also prints or notifies gives the information
a second, discoverable path; that redundancy is the accessibility story, not a
tooltip role. The presentation-only overlay is dismissed by `Esc` without
consuming the press and delivers one `overlay.dismissed` event with reason
`escape` or `view-closed`, so a recipe must treat dismissal as normal, not as an
error.

**Budget note.** Rung 2 is bounded and rate-limited by the host; a rejected or
rate-limited notification returns `false` or a typed denial, and the recipe
treats both as "not shown". Rung 3 paints at most 64 rows and 16 KiB per frame,
drops trailing rows deterministically, paints nothing under 10 columns while
keeping the session live, and admits one surface per window per generation.

### Example: detail fallback

```lua
-- Manifest grants ui.rich and platform.notify.
local function show_detail(text)
  local ok = bitty.notify.show({ title = "Detail", body = text, urgency = "low" })
  if not ok then
    return false -- rate-limited or declined: the caller keeps the inline suffix
  end
  return true
end

-- Manifest also grants ui.overlay and ui.present.
local function show_card(lines)
  local children = { { kind = "Text", text = "detail", fg = "accent", bold = true } }
  for i = 1, math.min(#lines, 4) do
    children[#children + 1] = { kind = "Text", text = lines[i] }
  end
  local handle = bitty.ui.mount("overlay", {
    kind = "Column",
    fg = "foreground",
    children = children,
  })
  return bitty.ui.present.show(handle) -- true: painted, no input authority
end
```

## C-4 Modal sheet

**Surface status: Not available.** This entry records the absence and the
supported substitutes; it is not a design for a plugin modal.

**Scene shape.** None. A modal sheet implies a backdrop, a focus trap, an owning
dismissal rule, and a stacking tier above everything else. Plugin API v1
provides no node, slot, field, or call that carries any of those, and the single
modal authority stays Core-owned. `register_panel` and `PanelProvider` are
excluded from v1 and wait on the terminal-side panel contract, so a sheet cannot
be smuggled in as a panel either.

**Style hooks.** Not applicable. The nearest available surface paints with the
band style vocabulary, but it is not modal.

**Interaction.** Three substitutes exist, each with a different authority:

| Substitute                               | Grants                                 | What it cannot do                                                      |
| ---------------------------------------- | -------------------------------------- | ---------------------------------------------------------------------- |
| Presentation-only overlay (`ui.present`) | draw above the focused view, no input  | no clicks, no keys, no focus; Core owns `Esc` dismissal and reports it |
| Capture overlay (`bitty.ui.overlay.*`)   | own the single transient input capture | no modality, no backdrop, no focus trap, no tier selection             |
| Panel (`PanelProvider`, candidate)       | host-managed panel with activities     | excluded from plugin API v1                                            |

A confirm-or-cancel flow therefore either uses the capture overlay and accepts
that the user can leave it with `Esc`, or it moves the decision into the terminal
where the terminal already owns input. It must not paint a scrim, claim
`modal_active`, or block input outside its capture session. The capture verbs are
`acquire`, `poll`, and `release`, but their argument spelling is not uniform
across the corpus: the pinned [bitty.ui reference](../sdk/reference/ui.md) mounts a
block and presents the handle, while the [palette plugin
pages](../docs/plugins/palette/schemas.md) show an options table. Confirm the
spelling against the target build instead of copying an example verbatim.

**Accessibility notes.** A capture overlay receives keyboard input and is the
only plugin surface that can, which makes it the only place where keyboard-only
use of a multi-step flow is possible today. It still has no focus trap and no
role metadata, so a destructive confirmation must not rely on modality for
safety. On the presentation-only surface, `Esc` dismisses without consuming the
press, so dismissal is a normal outcome and never a confirmation. No backdrop, no
`aria-modal` equivalent, and no announcement channel exists.

**Budget note.** One non-modal popup surface per window per generation; a second
claimant of the same surface fails closed with `E_UI_UNAVAILABLE` while the
incumbent is untouched. Capture is single-owner, capability-gated by
`ui.overlay.focus`, and revocable by Core.

### Example: bounded confirm without modality

```lua
-- Manifest grants ui.rich, ui.overlay, and ui.overlay.focus.
local sheet = bitty.ui.mount("overlay", {
  kind = "Column",
  children = {
    { kind = "Text", text = "Delete 3 files?", bold = true },
    { kind = "Text", text = "y confirm   n cancel   esc leaves" },
  },
})
-- Capture keys. This is input authority, not modality.
bitty.ui.overlay.acquire(sheet)
-- Read queued input, then always release, including after a dismissal.
bitty.ui.overlay.release(sheet)
```

## C-5 Status strip

**Surface status: Accepted v1.** Mounts into `bottom` under `ui.rich`. There is
no status slot: `statusline` was removed from the slot set, and status
presentation is a `bottom`-band composition.

**Scene shape.** A `Row` root of module atoms in declared groups. Modules are
independent: a module that cannot render emits no atom and no placeholder, and
each module hides only itself.

```text
Row
├── Row  workspaces   (capsules, see C-2)
├── Row  clock
└── Row  network · cpu · memory · battery   (present only when granted and available)
```

**Style hooks.** Per-module `fg`/`bg`/`bold`; no per-module container styling
and no separators. Modules join with a single literal space. The recipe may
emphasize one module with `bold` and one state with an accent fill, but it must
not encode availability in colour alone.

**Interaction.** Optional `on_click` per module bound to a provider-control
command; read and control are separate grants, so a clickable module that
acquires control must degrade to a non-clickable readout when only read is
granted. Omit the binding rather than binding a command that will be denied.

**Accessibility notes.** A status strip is a text row: its accessible content is
the concatenation of visible atoms, so each atom must read sensibly on its own
and in order (`cpu 12%`, not `12%`). Order carries meaning, so do not reorder
modules based on availability; hide and keep the rest in declared order.
Keyboard reachability is absent by construction, so any control a module exposes
needs a command entry outside the band.

**Budget note.** One block for the strip; one atom per module plus one per
workspace capsule. The strip is the surface most likely to grow with data, so
the recipe fixes the module list at activation and re-emits the whole row on
change rather than appending.

### Example: status strip

```lua
-- Manifest grants ui.rich and platform.cpu-read; no control grants.
local function module(name, text, args)
  return {
    kind = "Text",
    text = text,
    on_click = {
      command = "example-plugin:detail",
      args = { module = name },
      region = { id = "module:" .. name, kind = "module" },
    },
  }
end

local strip = bitty.ui.mount("bottom", {
  kind = "Row",
  children = {
    module("workspaces", " 1*  2 "),
    { kind = "Text", text = "  " },
    module("clock", "12:04"),
    { kind = "Text", text = "  " },
    module("cpu", "cpu 12%"),
  },
})
```

## Bar plugin mapping

The bar plugin is the reference implementation. The table records where its
current, registry-pinned behaviour matches a recipe, so the catalog can be
checked against a real surface instead of against itself. It is a mapping of
pinned behaviour, not a claim that the bar calls shared code: no shared recipe
library exists.

| Bar element                                         | Recipe | Match                                                                                                    |
| --------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------- |
| Root `Row` with left, center, and right groups      | C-1    | Exact: one root row, three anchored groups, empty groups omitted.                                        |
| Workspace capsules as padded filled `Text` atoms    | C-2    | Exact: one space of padding inside the atom, fill shared with padding, square corners, marks in order.   |
| Capsule marks and emphasis                          | C-2    | Exact: active is bold on an accent fill, inactive uses the selection fill, attention wins over occupied. |
| Inactive capsule focus binding                      | C-2    | Exact: `on_click` command with a bounded argument table.                                                 |
| `detail` command with notification fallback         | C-3    | Rungs 1-2 only: inline text and best-effort notification; no anchored tooltip.                           |
| Provider modules that hide when absent or ungranted | C-5    | Exact: per-module omission, single-space join, declared order.                                           |
| No modal surface                                    | C-4    | Consistent with the absence: the bar has no sheet and cannot acquire modality.                           |

What this mapping does **not** establish: the bar does not import these recipes
as code, so there is no dogfood proof of a shared implementation, and the
migration of the bar onto catalog entries remains open work in its own
repository. The bar's richer widget and unified-chrome directions are separate
candidates and are not recipes here.

## Security review

Recipes change nothing about the security posture; they are compositions of
already-admitted pieces. The properties that matter are still enforced by the
host:

| Concern                | Position                                                                                                                     |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Presentation not truth | Every recipe mounts a declarative subtree; none writes grid, cursor, mode, scrollback, or layout.                            |
| No hot-path execution  | Bindings dispatch command ids; no plugin VM runs on the input path, and no recipe polls per frame.                           |
| Bounded inputs         | Node, depth, text, block, and string budgets apply per generation; the notification rung is host rate-limited.               |
| Authority separation   | Draw authority (`ui.present`) never implies input authority (`ui.overlay.focus`); no recipe asks for both to make a tooltip. |
| No ambient geometry    | Region identities derive rectangles from painted spans and carry no coordinates; no recipe reads or sets absolute positions. |
| Fail closed            | An unavailable group, denied command, declined notification, or rejected mount leaves the rest of the surface unchanged.     |
| No modal authority     | No recipe paints a backdrop, claims modality, or traps focus; the single modal authority stays Core-owned.                   |

No capability, budget, or trust decision is proposed here, so no `P0` criterion
is affected. A security reviewer is required if a future revision admits a new
node kind, an anchored surface, or an input-bearing band.

## Verification plan

An accepted revision of this catalog would need at least:

1. `just check` with zero markdownlint, link, metadata, language, agents, and
   hygiene issues.
2. A mount test for each of C-1, C-2, C-3 (rungs 1-2), and C-5 using only the
   fields the entry lists, with no additional field accepted.
3. A rejection test for each pattern's excluded shape: `Image`, `Table`, `Rule`,
   bordered `Block`, a radius field, and a padding field must fail with
   `E_UI_COMPONENT_INVALID`.
4. A routing test proving C-2 and C-5 bindings dispatch from `top`/`bottom` and
   do not dispatch from an `overlay` block.
5. An omission test proving a missing module or group emits no atom and no
   placeholder, and that existing atoms are unchanged.
6. A fallback test proving C-3 prefers inline text, falls through to the
   notification rung when inline is truncated, and treats a declined
   notification as "not shown".
7. A negative test proving C-4 cannot be built: no backdrop, no modality claim,
   no focus trap, and, on the presentation-only surface, `Esc` dismissal that
   does not consume the press.
8. A budget test proving a strip that exceeds the node budget fails closed
   rather than truncating silently.
9. A dogfood test in the bar repository proving each mapped element is produced
   by the catalog entry it claims, before any "migrated" statement is made.

## Alternatives considered

| Alternative                                     | Trade-off                                                                  | Disposition                                                          |
| ----------------------------------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Add real component constructors (`Pill`, etc.)  | Familiar authoring surface, but changes the accepted node set.             | Rejected; v1 composes from four kinds, additions need a new surface. |
| Publish the recipes as a shared Lua library     | Removes duplication, but creates a versioned dependency with no owner yet. | Deferred; the `bitty-ui-core` direction is candidate and unowned.    |
| Document only the bar's source shape            | Cheap, but ties every plugin to one implementation's choices.              | Rejected; recipes state the shape and let tokens differ.             |
| Leave tooltips as an open question              | Honest but unhelpful; authors would invent hover surfaces.                 | Rejected; the fallback ladder records what exists and what does not. |
| Treat the presentation overlay as a modal sheet | Reuses an existing host, but transfers modal authority to plugins.         | Rejected; the surface is non-modal by contract and paint-only.       |

## Affected contracts

| Contract                                                                            | Effect                                                                         |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted) | Unchanged; the catalog composes only its accepted fields.                      |
| [bitty.ui reference](../sdk/reference/ui.md) (draft)                                | Supplies the style, binding, routing, and budget facts the entries cite.       |
| [Plugin UI Slot Inventory](plugin-ui-slot-inventory-candidate.md) (candidate)       | Supplies per-slot multiplicity and conflict rules the entries assume.          |
| [Lua UI Component Model](lua-ui-component-model-candidate.md) (candidate)           | Records the Level 2 library and accessibility direction these recipes precede. |
| [Cross-Plugin Composition](cross-plugin-composition-candidate.md) (candidate)       | Would own any future shared recipe package or service.                         |
| Theme Token Contract (candidate, terminal-side)                                     | Owns token naming and fail-closed rejection of unknown tokens.                 |

## Open points

These are candidate open items, not accepted questions:

1. Whether these recipes become a versioned `bitty-ui-core` library, and which
   repository would own it.
2. Whether a tooltip or anchored detail surface becomes a primitive, and what
   anchoring contract it would need.
3. Whether a plugin modal surface ever exists, and how it would relate to the
   single Core-owned modal authority.
4. The accessibility property schema (role, name override, state) and whether
   band atoms can carry any of it.
5. Whether padding, radius, border, width, and gap become style fields, which
   would change several entries from literal-character recipes to real layout.
6. Region-id stability across theme changes, and whether a region id may be
   reused after an update.
7. Per-slot aggregate bounds across plugins, which would bound a composing
   band's total size.
8. Whether the bar repository migrates onto catalog entries, and what evidence
   would count as dogfood proof.

## Acceptance criteria

1. Every entry uses only fields that exist in the accepted surface or is marked
   **Not available** with the missing field named.
2. No entry implies a new node kind, slot, capability, budget, or error code.
3. Each entry carries a scene shape, style hooks, interaction and accessibility
   notes, a budget note, and a headed example.
4. The bar mapping is stated as mapping of pinned behaviour, with the
   un-migrated implementation called out.
5. Review by a plugin-ecosystem category owner and a docs curator, with
   terminal-side coordination for any entry that touches host geometry or the
   overlay contract.
6. Acceptance happens through the owning contract (the UI component or slot
   inventory track), not by flipping this record's status.

## P0 Review Sign-off

Not applicable: no security boundary, capability, resource ceiling, or trust
decision changes; every recipe is a composition of already-admitted pieces and
narrows authoring rather than widening authority.

## References

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) —
  accepted slot set, node subset, handle model, and capability gates.
- [bitty.ui reference](../sdk/reference/ui.md) — style fields, theme tokens,
  interaction bindings, band-only routing, overlay capture, presentation-only
  overlay, and per-generation bounds.
- [Plugin UI Slot Inventory (Candidate)](plugin-ui-slot-inventory-candidate.md) —
  per-slot purpose, multiplicity, bounds, and conflict resolution.
- [Lua UI Component Model (Candidate)](lua-ui-component-model-candidate.md) —
  five-level component hierarchy, composition principle, and accessibility
  direction.
- [Cross-Plugin Composition (Candidate)](cross-plugin-composition-candidate.md) —
  host-mediated reuse rules for any future shared recipe package.
- [Plugin system](plugin-system.md) — presentation-not-truth boundary and
  extension levels.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md) —
  `RichBlock` replacement rule and scene bounds.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md) —
  panel identity, focus routing, and Core-owned modal authority.
- [Theme Token Contract (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/theme-token-contract-candidate.md) —
  token naming and unknown-token disposition.
- [Bar plugin design](../docs/plugins/bar/design.md) and
  [bar schemas and contracts](../docs/plugins/bar/schemas.md) — the reference
  implementation's pinned presentation, settings, and scene contract.
- [bitty.notify reference](../sdk/reference/notify.md) — the notification rung's
  payload, gate, and acceptance semantics.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) — resource
  ceilings no recipe may widen.
