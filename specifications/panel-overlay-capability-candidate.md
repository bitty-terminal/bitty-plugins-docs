---
title: Panel Overlay Capability (Candidate)
description: Candidate contract for the plugin-facing panel.overlay verb, covering the call shape, parameter grammar, borrowed and proposed bounds, typed denials, lifetime, and attribution to a panel generation
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: true
sidebar_order: 31
---

# Panel Overlay Capability (Candidate)

> Status: **draft candidate** — not **Accepted**, not **Verified**, not
> **Compatible**, and not normative. This document does **not authorize
> implementation**, ships no behavior, and grants no compatibility promise. It
> exists because
> [bitty-terminal/bitty#2476](https://github.com/bitty-terminal/bitty/issues/2476)
> residual 8 is blocked on a contract question: the Core can now admit an
> overlay that belongs to a plugin's panel, but no plugin-facing verb exists to
> raise one. The Core half landed (`bitty@3311892f`,
> `SceneHost::admit_attributed_overlay`, `crates/bitty-runtime/src/scene_host.rs:418`);
> the plugin half does not. `panel.overlay` is a capability string with no
> host backend, no grammar, no bound, and no error code.
>
> Every rule in this document is a **decision proposal**. "Proposes" means the
> text asks for review and acceptance; it never means the host implements it,
> and it never means the capability is usable. Where a rule is not a proposal
> it cites an existing accepted contract or an existing Core constant by file
> and line, and no rule here may widen a value, ceiling, or gate owned by
> another document.

## Purpose and scope

In scope:

- the plugin-facing call shape for raising, replacing, and dismissing one
  panel-attributed overlay, and the namespace it must live in;
- the parameter grammar: kind, tier, text, tooltip, and the optional
  bounds/anchor requests;
- multiplicity per panel, per plugin generation, and per window, and what
  happens when a plugin raises a second surface;
- the bounds a call is subject to, split into bounds **borrowed** from existing
  owners and bounds **proposed** here;
- typed denials and refusal semantics, including the rule that a refusal
  allocates nothing;
- lifetime and ownership: who dismisses, what a plugin generation change does,
  and how the surface composes with the three overlay owners that already
  exist;
- the capability mapping: which existing grants a raise consumes, and how the
  bar and statusline claim rules interact with a module that both composes
  status and raises overlays.

Out of scope, and each owned elsewhere:

- panel identity, panel creation, mounting, focus routing, and the panel-level
  budgets (accepted,
  [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md));
- `PanelProvider` registration, panel-to-view placement, and the panel
  lifecycle questions that stay open in the panel contract (`RFC-OQ-2`,
  `RFC-OQ-3`); this document answers only the `panel.*` capability mapping
  question (`RFC-OQ-5`) and only as a proposal;
- the closed slot vocabulary, the v1 `SceneNode` subset, the `block_id` handle
  model, and the `ui.rich`/`ui.overlay`/`ui.present`/`ui.overlay.focus` gates
  (accepted, [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md));
- the overlay focus and transient input-capture surface `bitty.ui.overlay.*`
  (accepted, `W-01`); this candidate adds no input authority and no verb to
  that namespace;
- the capability identifier grammar, manifest schema, and deny-by-default grant
  lifecycle (accepted, [Plugin Platform RFC](plugin-platform-rfc.md));
- per-plugin instruction, memory, task, and queue ceilings (accepted,
  [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md));
- the declarative scene schema and its limits (accepted,
  [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md));
- the draw-list contract for panel content (candidate,
  [Plugin Drawing Capability](plugin-drawing-rfc-candidate.md)), which this
  document neither depends on nor amends.

## The gap this document addresses

Four facts define the gap. All four are readable in the canonical `bitty`
repository at `3311892f75b38adb50e835822e1129c72866a6a3`; none of them is a
contract decision.

1. **The registry can write an attributed overlay.** `PanelRegistry::create_panel_overlay`
   (`crates/bitty-runtime/src/registry/panel.rs:2708`) validates the
   `(panel_id, generation)` pair through `get_panel` (`:1735`) **before** it
   allocates, delegates the admission to `create_overlay` (`:2679`), and only
   then records the attribution. A stale or unknown generation therefore cannot
   claim a surface, and a refused admission leaves no attribution row behind.
2. **The scene host can admit exactly one of them for a plugin owner.**
   `SceneHost::admit_attributed_overlay` (`crates/bitty-runtime/src/scene_host.rs:418`)
   resolves the owner's generation-owned panel (`panel_handle_for`) and admits a
   single `OverlayKind::NonModal` overlay with an empty rect and empty text.
   Its documented refusal path is explicit: `E_UI_UNAVAILABLE` when the host
   owns no panel for that owner, and **"a refusal leaves nothing allocated"**.
3. **The only caller is Core, and it is not reachable from a plugin.**
   `PresentationOverlayAttribution`
   (`crates/bitty-runtime/src/plugin_runtime/presentation_overlay.rs`) is the
   sole consumer. Its `admit`/`release` methods are host-only, and
   `admit_attributed_overlay` is `pub(crate)`. No bridge function calls it, so
   no Lua code can reach the writer that already exists.
4. **The capability identifier exists and nothing else does.** `panel.overlay`
   appears in the closed `CapabilityFamily::Panel` list
   (`crates/bitty-plugin-host/src/capability.rs:217`) and carries the consent
   description "Show panel overlays and modals" (`:547`). It also appears in the
   manifest validation table (`crates/bitty-package/src/manifest.rs:417`) and in
   the `PanelLike` consent profile
   (`crates/bitty-plugin-host/src/consent_profile.rs:110`). There is no
   `bitty.panel.overlay*` bridge function, no parameter validation, no bound
   enforced at a plugin boundary, and no plugin-visible error code of its own.

Two consequences follow, and they shape the rest of this document.

- **Every failure on the current path collapses to one code.** `panel_error`
  (`crates/bitty-runtime/src/scene_host.rs:600`) maps every `PanelError`
  variant, including the overlay-budget refusal, to
  `E_UI_UNAVAILABLE` (aliased as `E_SCENE_UNAVAILABLE` at `:83`). That is
  adequate for a host-internal caller that only needs to know the surface did
  not get attributed. It is not adequate for a plugin, which cannot tell a
  missing grant from an exhausted envelope from a stale panel id.
- **The plugin-facing surface is genuinely undefined.** The identifier's
  description says "overlays and modals", while the Core writer admits only
  `NonModal`, and the accepted design keeps the single modal authority
  Core-owned. Resolving that mismatch is part of what this candidate proposes;
  it is not something the code has already decided.

This document does **not** claim acceptance. The `panel.*` capability mapping is
open question `RFC-OQ-5` of the panel contract, owned by that contract's owner.
What follows is candidate text offered for that review, in the plugin corpus
where the verb would live.

## Normative sources this specification must not weaken

- [Plugin Platform RFC](plugin-platform-rfc.md) (accepted): deny-by-default
  capability grammar with no wildcards, the grant lifecycle including
  manifest-hash binding and revocation at the next dispatch boundary, the rule
  that presentation is never Terminal Truth, the rule that there are no
  hot-path events, and the three-level queue budgets with `DropOldest` as the
  accepted v1 default.
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (accepted): the closed seven-name slot set, the accepted overlay wording (slot
  content never claims focus; the Core-owned capture surface is
  `bitty.ui.overlay.*` gated by `ui.overlay.focus`; presentation of a mounted
  block is `bitty.ui.present.*` / `bitty.ui.overlay.show`/`hide` gated by
  `ui.present`), and the stable v1 error vocabulary.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (accepted):
  RC-1 through RC-11 ceilings and FS-1 through FS-9 failure semantics.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  (accepted): the `4+1` overlay envelope, `OverlayBusy` on a second modal, the
  rule that overlay geometry derives from the `Window` logical area rather than
  a panel rect, and the rule that an overlay never mutates grid, scrollback, or
  view attachment.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  (accepted): SCN-1 through SCN-5 and the replacement-by-`BlockId` rule.
- [Overlay Ownership and Composition Reconciliation](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/overlay-ownership-reconciliation.md)
  (draft, terminal corpus): the three overlay owners, the `OverlayTier` and
  `OverlayKind` vocabularies, and the single modal authority. This candidate
  composes with those owners and creates no fourth system.
- [bitty.ui Reference](../sdk/reference/ui.md) (draft reference): the current
  slot hosts, the ordered mount checks, and the typed denial table this
  document's error style follows.

## Terminology

| Term               | Meaning in this document                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| Attributed overlay | One registry overlay row whose lifetime is owned by a `(panel_id, generation)` pair and dismissed when that generation releases.     |
| Raising plugin     | The plugin generation on whose behalf a surface is admitted; the only identity the surface is attributed to.                         |
| Overlay envelope   | The accepted per-window bound of at most one modal plus a bounded non-modal count, enforced by `OverlayManager`.                     |
| Plugin kind        | The subset of the accepted `OverlayKind` set a plugin may request: `nonmodal` and `tooltip`.                                         |
| Borrowed bound     | A value owned by another document or by a Core constant, cited rather than restated as this document's own number.                   |
| Proposed bound     | A new value this candidate asks review to accept; labelled **proposed** at every appearance and never presented as an accepted fact. |
| Refusal            | A typed denial that allocates nothing: no overlay id, no registry row, no envelope slot, no attribution row.                         |

## Proposed verb surface

### Namespace, and why it is not `bitty.ui.overlay.*`

The accepted surface already spends `bitty.ui.overlay.*` on two distinct
meanings, and neither is this one:

- **Input capture.** `bitty.ui.overlay.acquire / update / release / show / hide / poll`,
  gated by `ui.overlay.focus` (v2, `W-01`), with a single Core-owned capture
  switch and revocation on cancel, submit, focus switch, unload, crash, and
  Core-side timeout.
- **Presentation of a mounted block.** `bitty.ui.overlay.show(handle)` /
  `bitty.ui.overlay.hide(handle)`, alongside `bitty.ui.present.show` /
  `bitty.ui.present.hide`, gated by `ui.present`, presenting a block already
  mounted into the `overlay` slot as a non-modal `Popup`.

Adding a third meaning to that namespace would make one prefix mean "capture
input", "show a block I mounted", and "raise a free-standing surface". The
candidate therefore uses the namespace that already exists for panel verbs.
`bitty.panel.*` ships `create`, `close`, `destroy`, `get_presentation`,
`set_presentation`, `toggle_floating`, `set_pinned`, and `get_state`
(`crates/bitty-lua/src/host.rs`, `panel_*` host methods; parity asserted in
`crates/bitty-lua/tests/lua_parity.rs`), all with flat snake-case names. The two
proposed verbs follow that shape.

```lua
bitty.panel.overlay_show(panel_id, opts) -> overlay_id
bitty.panel.overlay_hide(panel_id, overlay_id) -> boolean
```

Both verbs take the calling plugin's own `panel_id`, which is the handle a
plugin already holds from `bitty.panel.create`. Neither verb accepts a
`generation` argument: the host resolves the calling generation and validates
the pair itself, exactly as `create_panel_overlay` does today. A plugin can
therefore never name another generation's row, and cannot forge an attribution.

### Parameter grammar

`opts` is a table. Unknown fields are refused rather than ignored, matching the
closed-value discipline the bridge already applies to `ui.mount` components.

| Field     | Type   | Domain                                            | Required | Default                 |
| --------- | ------ | ------------------------------------------------- | -------- | ----------------------- |
| `kind`    | string | `nonmodal`, `tooltip` (closed plugin kind domain) | yes      | —                       |
| `text`    | string | at most 128 characters                            | yes      | —                       |
| `tooltip` | string | at most 256 characters                            | no       | absent                  |
| `tier`    | string | `editor`, `float`, `popup`, `messages`            | no       | host default for `kind` |
| `bounds`  | table  | `{ x, y, width, height }`, integers               | no       | host placement          |
| `anchor`  | string | `panel`, `focus`                                  | no       | `panel`                 |

Rules:

1. **The kind domain is the accepted set minus the authority kinds.**
   `OverlayKind` is `Modal`, `NonModal`, `Tooltip`, `Palette`
   (`crates/bitty-ui/src/panel.rs:230`). A plugin may request `nonmodal` and
   `tooltip` only. `Modal` is refused because the single modal authority is
   Core-owned and `modal_active` is what the app's capture predicate reads; a
   plugin that could set it would be choosing whether keys are captured.
   `Palette` is refused because a palette is a Core-owned transient search
   surface with its own focus and dismissal semantics. Both refusals are
   validation denials, not capability denials: the request is outside the
   plugin domain of a closed parameter, not a missing grant.
2. **`tier` names a paint band and is never a stacking authority.**
   `OverlayTier` is `Editor < Float < Popup < Messages`
   (`crates/bitty-ui/src/layout.rs:42`). A requested tier is validated against
   that set, then resolved by host policy against the kind's ceiling:
   `nonmodal` resolves no higher than `Popup` (the accepted presentation tier,
   `PRESENTATION_OVERLAY_TIER`), `tooltip` no higher than `Messages`. A request
   above the ceiling, or a tier the host does not host, is refused. A plugin
   never selects an order relative to another plugin's surface; same-tier order
   stays construction order, owned by the geometry owner.
   **Proposed:** the current registry `Overlay` row carries no tier field
   (`crates/bitty-ui/src/panel.rs:239`), so tier is a request the host may only
   honour at or below the kind ceiling until the registry row records one. See
   [OP-2](#open-points).
3. **Text is refused, not truncated.** The accepted implementation truncates
   overlay text at a character boundary at `MAX_OVERLAY_TEXT_LEN` (128) and
   tooltips at `MAX_OVERLAY_TOOLTIP_LEN` (256), setting a `truncated` flag
   rather than failing (`crates/bitty-ui/src/panel.rs:224`, `:262`).
   **Proposed:** the plugin verb refuses a longer `text` with a typed denial
   instead of silently painting a shortened string. This is stricter than the
   accepted constant — it never widens it — and a plugin-facing API that
   quietly changes the content a plugin authored is a worse contract than one
   that says no. Host-internal rows keep the truncation behavior unchanged.
4. **Bounds are a request inside the window logical area.** Overlay geometry is
   derived from the `Window` logical area, not from a panel rect (accepted,
   Panel Runtime RFC), and the Core writer currently passes an empty rect
   because the present loop owns the geometry. **Proposed:** a supplied
   `bounds` is validated as integers inside the window logical area, may be
   narrowed by host policy, is never expanded beyond the request, and an
   out-of-area or non-positive request is refused rather than clamped. Whether
   plugins may choose their own geometry at all is [OP-3](#open-points); the
   default is host placement.
5. **Anchors are closed and view-scoped, not pixel-scoped.** `panel` binds the
   surface to the raised panel; `focus` binds it to the focused view, matching
   the presentation session's focus-follow behavior. There is no block-id
   anchor, no cell anchor, and no pointer anchor: the registry row has no field
   for one, and overlay hit-testing and pointer routing stay undefined by the
   accepted surface.
6. **Every value is validated before anything is allocated.** The order is
   fixed in [Typed denials and refusal semantics](#typed-denials-and-refusal-semantics).

### Multiplicity

| Scope                           | Bound                                                       | Owner                |
| ------------------------------- | ----------------------------------------------------------- | -------------------- |
| Per `(panel_id, generation)`    | 1 live attributed surface (**proposed**, OV-1)              | this document        |
| Per plugin generation           | 2 live surfaces, one of them a tooltip (**proposed**, OV-2) | this document        |
| Per window, non-modal           | `MAX_OVERLAYS_PER_WINDOW` = 4 (accepted)                    | `OverlayManager`     |
| Per window, modal               | 1, never consumed by a plugin (accepted)                    | `OverlayManager`     |
| Per window, plugin presentation | 1 presentation overlay (accepted, `ui.present`)             | presentation session |

Rules:

1. **A second raise for the same panel supersedes.** It dismisses the live
   surface for that panel and admits a new one under a new id; the old id
   becomes stale and `overlay_hide` on it returns `false`. Supersession is
   counted, so a raise loop is diagnosable. **Proposed:** the superseded row
   queues one `overlay.dismissed` observation with a `superseded` reason. The
   accepted reasons are `escape` and `view-closed` only, so this reason is an
   additive vocabulary item and needs review.
2. **A plugin's surfaces never stack.** Because OV-1 is 1, a plugin cannot use
   repeated raises to build a stack inside its own panel; the newest surface
   wins and the older one is gone, which is the same "newest handle wins"
   rule the presentation session already applies.
3. **Raises share the window envelope with everyone else.** Two plugins raising
   one surface each consume two of the four accepted non-modal slots. When the
   envelope is full the raise is refused with a budget denial naming the bound;
   the manager never evicts another owner's live surface to make room.
4. **A plugin never consumes a modal slot.** No plugin verb can set
   `modal_active`, so a plugin cannot deny keys to another plugin by raising a
   modal, and the single modal authority keeps exactly one owner.

### What a declined call returns

A refused `overlay_show` **raises** a catchable Lua error table
(`class` / `code` / `message`, per `BridgeError::to_error`) and returns no
value. Specifically, a refusal:

- mints no `overlay_id`;
- creates no registry overlay row and therefore consumes no envelope slot;
- writes no attribution row;
- emits no `overlay.dismissed` observation, because nothing was shown;
- increments the per-generation refusal counter so the refusal is observable
  rather than silent.

`overlay_hide` does not raise for an unknown, stale, foreign, or already
dismissed id: it returns `false` and changes nothing. That asymmetry is
deliberate and copies two existing decisions — `dismiss_attributed_overlay` is
documented idempotent, and the accepted `hide` is ungated so that teardown never
wedges. A plugin's cleanup path must never be able to fail.

## Bounds

### Borrowed bounds

These values are owned elsewhere. They are cited so that a reviewer can see the
whole envelope a raise sits inside, and no rule in this document changes any of
them.

| Dimension                                | Value                            | Owner                                                                |
| ---------------------------------------- | -------------------------------- | -------------------------------------------------------------------- |
| Overlay envelope per window              | 1 modal plus 4 non-modal (`4+1`) | Panel Runtime RFC §Overlay (4+1); `crates/bitty-ui/src/panel.rs:224` |
| Overlay text per surface                 | 128 characters                   | `MAX_OVERLAY_TEXT_LEN`, `crates/bitty-ui/src/panel.rs:225`           |
| Overlay tooltip per surface              | 256 characters                   | `MAX_OVERLAY_TOOLTIP_LEN`, `crates/bitty-ui/src/panel.rs:226`        |
| Scene nodes per `RichBlock`              | 2048                             | SCN-1, Rich Presentation RFC                                         |
| Scene tree depth per block               | 32                               | SCN-2, Rich Presentation RFC                                         |
| Text bytes per block                     | 256 KiB                          | SCN-3, Rich Presentation RFC                                         |
| Aggregated rich bytes per terminal       | 2 MiB                            | SCN-4, Rich Presentation RFC                                         |
| Blocks per terminal                      | 64                               | SCN-5, Rich Presentation RFC                                         |
| Semantic snapshot payload                | 256 KiB                          | `SNAPSHOT_MAX_BYTES`, `../sdk/plugin-api-v1-lua-surface-rfc.md`      |
| Per-subscription event queue             | 64 events                        | RC-5, `../runtime/isolation-resource-rfc.md`                         |
| Per-plugin event queue                   | 1024 events / 256 KiB aggregate  | RC-5, `../runtime/isolation-resource-rfc.md`                         |
| Pending `overlay.dismissed` observations | 64, drop-oldest beyond           | `PENDING_DISMISSED_MAX`, `plugin_runtime/presentation_overlay.rs`    |
| Callback CPU and wall time for a raise   | 10^7 VM instructions / 50 ms     | RC-1, `../runtime/isolation-resource-rfc.md`                         |
| Plugin VM memory                         | 32 MiB accounted                 | RC-2, `../runtime/isolation-resource-rfc.md`                         |

Two notes keep this table honest:

- **The SCN limits bind the transport, not this first slice.** A text-and-tooltip
  overlay creates no `RichBlock`; SCN-1 through SCN-5 are listed because a
  surface that ever carries scene content must obey them, and because a reviewer
  must be able to see that nothing here proposes a wider block budget. The
  2 MiB and 64-block rows are per-terminal aggregates, so a panel overlay cannot
  use them to escape the per-block caps.
- **The queue bounds bound the refusal path, not the raise.** A plugin that
  raises and hides in a tight loop fills the bounded `overlay.dismissed` buffer
  with drop-oldest; it cannot grow the buffer, and it cannot make Core call back
  into it.

### Proposed bounds

Each row is **proposed** — a new number this candidate asks review to accept, not
a restatement of an accepted constant. On acceptance these become owned values
that change only by revision of this document, and every implementation test
must parameterize on them rather than hard-code them.

| ID   | Dimension                                             | Proposed value                   | Rationale                                                                                                                                                                                                    | Failure                  |
| ---- | ----------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------ |
| OV-1 | Live attributed surfaces per `(panel_id, generation)` | 1                                | **Proposed.** The Core writer admits one surface per owner and generation release dismisses by exactly that key; two live rows would make supersession and dismissal ambiguous.                              | `E_PANEL_OVERLAY_BUDGET` |
| OV-2 | Live surfaces per plugin generation                   | 2 (one surface plus one tooltip) | **Proposed.** A tooltip is a second surface in practice; a cap of 2 leaves at least two of the four accepted non-modal slots for other plugins.                                                              | `E_PANEL_OVERLAY_BUDGET` |
| OV-3 | `overlay_show` admissions per plugin generation       | 8 per second, coalesced          | **Proposed.** A raise/hide loop inside one command callback would otherwise churn the registry; 8/s stays below the accepted 10 events/s metadata rate (RC-8) and inside RC-1's 50 ms wall budget.           | `E_PANEL_OVERLAY_BUDGET` |
| OV-4 | Requested bounds area                                 | 4096 cells, no axis above 256    | **Proposed.** Matches the offscreen-surface area in the drawing candidate (DR-6) so one overlay can never request more paint area than a bounded surface, keeping the two presentation proposals comparable. | `E_PANEL_OVERLAY_BUDGET` |

Two invariant rules hold regardless of the numbers:

- **Never above the owner.** No OV value may raise an RC ceiling, an SCN limit, a
  panel budget, or the `4+1` envelope. The effective bound is the minimum of
  every applicable one, and a lower layer can only tighten.
- **Attributed or absent.** Every ceiling is enforced per plugin generation and
  emits observable accounting. Enforcement that cannot run means the raise is
  refused, not that it runs unbounded (FS-7).

## Typed denials and refusal semantics

### Check order

A raise is checked in this order, and each step fails closed before the next
runs. The shape follows the accepted `ui.mount` ordering, which is the corpus's
existing precedent for a bridge call with several gates.

1. **Bridge shape.** `panel_id` is an integer, `opts` is a table, fields have
   the declared types, and no unknown field is present.
   `E_PANEL_OVERLAY_INVALID`.
2. **Pre-commit expiry.** The call runs through the bounded mutation guard, so
   an expired call fails without mutating. `E_TIMEOUT`.
3. **Capability.** `panel.overlay` and `ui.overlay` are granted to this
   generation and not revoked. `E_CAPABILITY_DENIED`.
4. **Ownership.** The named panel exists and belongs to the calling generation;
   the plugin kind is inside the closed plugin domain; the tier is inside the
   closed set and at or below the kind ceiling; `text` is within 128 characters
   and `tooltip` within 256; `bounds`, if present, is positive integers inside
   the window logical area. `E_PANEL_OVERLAY_NOT_OWNER` or
   `E_PANEL_OVERLAY_INVALID`.
5. **Host availability.** This build hosts panel overlays, the runtime is not in
   safe mode, and the panel still has a host surface. `E_PANEL_OVERLAY_UNAVAILABLE`.
6. **Envelope and rate.** OV-1 through OV-4 and the accepted `4+1` envelope.
   `E_PANEL_OVERLAY_BUDGET`.

Only after all six steps does the host allocate: mint the id, write the registry
row, charge the envelope slot, and record the attribution. There is no step that
allocates first and validates second.

### Codes

The codes below are proposed additions in the repository's existing
`E_<AREA>_<CONDITION>` style. The two reused codes are accepted and unchanged.

| Code                          | Class        | When                                                                                                                                                                                                                                    |
| ----------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `E_CAPABILITY_DENIED`         | `runtime`    | Accepted code. `panel.overlay` or `ui.overlay` is absent, revoked, or not declared in the manifest.                                                                                                                                     |
| `E_TIMEOUT`                   | `budget`     | Accepted code. The call expired before commit.                                                                                                                                                                                          |
| `E_PANEL_OVERLAY_INVALID`     | `validation` | **Proposed.** Malformed field or type, an unknown or reserved `kind`, an unknown or too-high `tier`, `text` over 128 characters, `tooltip` over 256, or a `bounds` value that is not a positive integer inside the window logical area. |
| `E_PANEL_OVERLAY_UNAVAILABLE` | `runtime`    | **Proposed.** No panel-overlay backend in this build, `bitty --safe`, or the named panel no longer has a host surface.                                                                                                                  |
| `E_PANEL_OVERLAY_NOT_OWNER`   | `validation` | **Proposed.** The `panel_id` is not owned by the calling generation, or `overlay_id` is not a surface this panel currently holds.                                                                                                       |
| `E_PANEL_OVERLAY_BUDGET`      | `budget`     | **Proposed.** OV-1, OV-2, OV-3, OV-4, or the accepted `4+1` envelope is exceeded.                                                                                                                                                       |

### Refusal rules

1. **A refusal allocates nothing.** No id, no registry row, no envelope slot, no
   attribution row, no observation. This is the Core writer's own documented
   contract ("a refusal leaves nothing allocated"), restated as the plugin-facing
   requirement rather than invented here.
2. **A refusal is typed, never a falsy success.** `overlay_show` raises. Only
   `overlay_hide` reports `false`, and only because teardown must never fail.
3. **A refusal is attributed.** The message names the panel, the generation, and
   the bound or gate that refused, so one plugin's diagnostics can distinguish
   its own exhaustion from another owner holding the envelope.
4. **A refusal is counted.** Each generation carries a refusal counter, mirroring
   the `attribution_refused` counter the presentation session keeps. A refused
   raise is visible in diagnostics; it is never silent.
5. **A refusal is not partial.** The row, its slot, and its attribution commit
   together or not at all. A failure between allocation and attribution must not
   leave an unattributed overlay with no dismissal path.

## Lifetime and ownership

### Who dismisses

| Path                   | Actor                     | Effect                                                                                                                                                  |
| ---------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `overlay_hide`         | The raising plugin        | Dismisses the row, releases its attribution, frees the envelope slot. Returns `false` when it is already gone.                                          |
| Supersession           | The raising plugin        | A second raise for the same panel dismisses the previous row and admits a new one under a new id.                                                       |
| Generation release     | The host                  | `release_generation_scope` dismisses every row attributed to `(panel_id, generation)` through `release_attributed_overlays` (`registry/panel.rs:2742`). |
| Allocation-epoch sweep | The host                  | Panel disposal dismisses every row attributed to the panel id (`sweep_allocation_epoch_families`, `registry/panel.rs:2765`).                            |
| Suspend, unload, crash | The host                  | Treated as a generation release; no surface survives its generation.                                                                                    |
| `Esc` or view close    | Core, on the user's input | Dismisses the surface and queues one `overlay.dismissed` observation. No plugin code runs on the input path.                                            |

Rules:

1. **The plugin is never called back on dismissal.** It may observe
   `overlay.dismissed` (`{ owner, handle, reason }`) on its next tick, and it may
   `overlay_hide` with an id it no longer holds. There is no dismissal callback,
   consistent with the accepted rule that there are no hot-path events.
2. **Idempotence is required at every path.** A dismissal that runs after the
   generation release already dismissed the row is a no-op, not a double count.
3. **A generation change retires the surface.** A new plugin generation owns new
   panel records; every row attributed to the previous generation is dismissed at
   its release. A stale generation's `panel_id` fails `E_PANEL_OVERLAY_NOT_OWNER`
   rather than rebinding to a live panel, which is what `get_panel` enforcing the
   allocation epoch already guarantees.
4. **A dismissed id is never reused.** Newest `show` mints a new id; an old id
   never addresses a new surface, so a plugin cannot confuse its own teardown.

### Composition with the three existing overlay owners

This candidate adds no fourth overlay system. It composes with the owners the
terminal corpus already records.

| Owner                                           | What it owns                                                     | What a plugin raise may do                                                                                                      |
| ----------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Geometry (`LayoutNode::Overlay`, `OverlayTier`) | Bounds and paint-tier order, `Editor < Float < Popup < Messages` | Request a tier at or below the kind ceiling and a bounds inside the window logical area; never set order against another owner. |
| Stacking (`OverlayManager`)                     | The live set, the `4+1` envelope, and the single modal authority | Occupy one non-modal slot as an ordinary row; never be modal, never evict another owner, never touch `modal_active`.            |
| Presentation mode (`PresentationMode`)          | The requested display mode of one leaf                           | Have no interaction at all: a surface is not a display mode and never changes a leaf's mode.                                    |

The composition rules that follow:

1. **A surface is one row, not one system.** It is admitted through the same
   `OverlayManager` that bounds every other overlay, so the envelope is enforced
   once, in one place.
2. **Ordering is host-owned.** Two plugins' surfaces on the same tier resolve by
   construction order, exactly as the reconciliation record states; a plugin
   cannot request "above" another plugin.
3. **Presentation stays presentation.** A surface never mutates grid cells,
   cursor position, modes, scrollback, selection, or view attachment, and has no
   path to do so. It also never becomes Terminal Truth in an observation payload.

### When the plugin owns no terminal-slot scene panel

The Core writer already refuses this case: `admit_attributed_overlay` returns
`E_UI_UNAVAILABLE` when the host owns no panel for that owner, and the
presentation session counts the refusal while still presenting the surface
unattributed, because Core itself owns that surface's teardown.

The plugin path must behave differently, and this is a deliberate proposal:

- A plugin-raised surface is **refused** when the named `panel_id` is not owned
  by the calling generation — including the case where the generation never
  mounted a terminal-slot scene and therefore has no panel to attribute to. The
  denial is `E_PANEL_OVERLAY_NOT_OWNER`, and nothing is allocated.
- There is **no unattributed fallback**. An unattributed plugin surface would
  have no row for a generation release to dismiss and no owner for the epoch
  sweep to find, which is exactly the leak the attribution exists to prevent.
- The refusal is counted per generation, so a plugin that repeatedly raises
  against a panel it does not own is diagnosable rather than silently dropped.

A plugin that wants an overlay without owning a panel is out of scope here; the
`ui.present` path presents a block the plugin already mounted into the `overlay`
slot and is gated separately. Whether that path should ever be reachable without
a panel is a panel-contract question, not this document's.

## Capability mapping

| Identifier         | Status                          | What it gates here                                                                                                  |
| ------------------ | ------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `panel.overlay`    | Accepted identifier, no backend | **Required** for both proposed verbs. Deny-by-default; no wildcard; never implied by another panel identifier.      |
| `ui.overlay`       | Accepted                        | **Required**, because a surface paints through the same overlay host whose slot gate is `ui.overlay`.               |
| `ui.rich`          | Accepted                        | Not required for a text-and-tooltip surface; required only if a later revision lets a surface carry scene content.  |
| `ui.overlay.focus` | Accepted (v2)                   | Never required, never implied, never sufficient. Input capture stays a separate grant on the same host.             |
| `ui.present`       | Accepted                        | Neither implies nor is implied. Presenting a mounted block and raising a free-standing surface are different verbs. |
| `panel.create`     | Accepted                        | Required to obtain the `panel_id` the verbs take, and never implied by `panel.overlay`.                             |
| `panel.focus`      | Accepted                        | Unrelated to raising; focus routing stays with the panel contract.                                                  |

Rules:

1. **`panel.overlay` alone is not enough.** A generation with `panel.overlay` but
   no `ui.overlay` is refused with `E_CAPABILITY_DENIED`, matching the accepted
   rule that overlay-slot content requires both heads. Requiring both keeps one
   grant from becoming two authorities by implication.
2. **`panel.overlay` implies no input authority.** It grants no capture, no key
   handling, no pointer routing, and no focus change. The accepted `W-01` grant
   remains the only path to input, and a surface raised by this verb takes no
   input at all.
3. **No identifier is widened.** The candidate adds no capability string; it
   gives a backend to one that already exists. If review decides a separate head
   is needed for tooltip-class surfaces, that is an additive identifier and a
   revision of the accepted grammar, not a local choice here.

### Bar and statusline claims

The claim rules are the accepted ones, and a raise neither satisfies nor bypasses
them:

1. **Status composes on `bottom`.** `statusline` is not a slot; status
   presentation is ordinary `bottom`-band subtrees gated by `ui.rich`. A module
   that composes status and raises an overlay needs `ui.rich` for the mount and
   `panel.overlay` plus `ui.overlay` for the raise, and the two contributions are
   counted separately.
2. **`tabline` is an exclusive claim.** A `tabline` mount without the manifest's
   exclusive `[lazy].claims` entry fails with `E_UI_CLAIM_REQUIRED`. Raising an
   overlay grants no claim, consumes no claim, and never displaces the incumbent
   claimant.
3. **An overlay never composes into a band.** The surface sits above the band
   area as one overlay row; it takes no band row, reserves no column, and does
   not shrink the layout container the way a band does. A module's overlay
   therefore cannot be used to paint over a band it is not entitled to.
4. **Bar ownership is unchanged.** Core draws no workspace display of its own and
   reserves no band, so a raise cannot "claim the bar" as a side effect.

## Not in scope / depends on

Nothing below is settled by this document, and no implementation may treat a
decision here as settling one of them.

1. **Panel provider registration and placement.** `PanelProvider` registration
   (`RFC-OQ-2`) and panel-to-view placement (`RFC-OQ-3`) stay with the panel
   contract. This candidate assumes only that a plugin can hold a `panel_id` it
   owns and that a generation release dismisses what that panel owns.
2. **The `panel.*` capability mapping itself.** This document is candidate text
   for `RFC-OQ-5`. Until that question is answered, `panel.overlay` remains an
   identifier with no backend, which is the state the code is in today.
3. **Scene content inside an overlay.** Whether a surface may carry a
   `SceneNode` subtree, and therefore whether SCN-1 through SCN-5 apply to it,
   is a question for the overlay host and the rich-presentation contract. This
   candidate carries text and a tooltip only.
4. **Pointer and hit-test routing.** Overlay pointer routing has no accepted
   geometry contract and stays deferred; a raised surface takes no pointer input.
5. **The per-slot aggregate bound.** The slot inventory's aggregate bound per
   slot is itself open; this document bounds panel overlays per generation and
   per window and does not fix a slot aggregate.
6. **Theme tokens and contrast.** A surface resolves style against host tokens
   owned by the appearance contracts; this document defines no color and no
   contrast rule.

## Open points

1. **OP-1 — Text over the accepted cap: refuse or truncate?** This candidate
   proposes refusal so a plugin is never quietly painted differently from what it
   submitted. The alternative — keep the accepted truncation and surface the
   `truncated` flag through the return value — is cheaper and matches the
   existing row behavior. Either answer must be chosen once, for both verbs.
2. **OP-2 — Does the registry `Overlay` row gain a tier field?** The row records
   id, kind, bounds, text, tooltip, generation, and `truncated`
   (`crates/bitty-ui/src/panel.rs:239`), and no tier. Until it records one, a
   requested tier can only be checked against the kind ceiling and then resolved
   by host policy, which makes the parameter advisory. Making it authoritative is
   a Core change and belongs in that review.
3. **OP-3 — May a plugin choose its own geometry?** The accepted rule derives
   overlay geometry from the window logical area and keeps placement host-owned.
   This candidate therefore treats `bounds` as a request that host policy may
   narrow. If review wants plugin-chosen geometry, the geometry owner's contract
   has to say so first.
4. **OP-4 — Is the `superseded` dismissal reason accepted?** The accepted
   reasons are `escape` and `view-closed`. Supersession needs a third, or it must
   reuse `escape`, which would be a false statement about what happened.
5. **OP-5 — Is a query verb needed?** A read-only `overlay_state(panel_id)`
   returning the live surface's kind, tier, and id would make idempotent teardown
   easier and diagnostics cheaper. It is omitted from the first slice to keep the
   surface at two verbs; adding it later is additive.

## Security review

This proposal creates no ambient authority and relaxes nothing it cites.

| Element                                                    | Gate it serves                                                 | Threat/risk |
| ---------------------------------------------------------- | -------------------------------------------------------------- | ----------- |
| Raise is a data call; no callback into the plugin          | Invariant 4 (no hot-path execution); accepted no-hot-path rule | T-07        |
| Modal and palette kinds refused to plugins                 | Single modal authority; capture predicate stays Core-owned     | T-06, T-13  |
| Ownership validated before allocation                      | No confused-deputy surface ownership; no cross-generation ids  | T-06        |
| Refusal allocates nothing and is counted                   | Bounded inputs; no partial state; no silent failure            | T-06, T-07  |
| Shared `4+1` envelope, no eviction of another owner        | Availability isolation between plugins                         | T-07, T-13  |
| Presentation-only; never grid, cursor, scrollback, or mode | Invariant 3 (presentation, never Terminal Truth)               | T-13        |
| No input, no capture, no pointer routing implied           | Invariant 2 (no ambient authority)                             | T-06        |

Residual risks a reviewer must weigh:

- **The refusal path is the security surface.** An attributed surface that is
  admitted before its attribution row is written would be a surface nobody
  releases. The order of operations, not the bound values, is what protects the
  envelope.
- **The kind domain is a policy boundary, not a type boundary.** `Modal` is a
  valid `OverlayKind`, so a future refactor that validates only "is a known kind"
  would silently grant modal authority. The refusal must be tested as its own
  case.
- **Tier is advisory until the row records one** ([OP-2](#open-points)). A
  reviewer should not accept a tier parameter as meaningful while the storage
  cannot hold it.

No `P0` criterion is modified by this document. A security reviewer is required
for the kind domain, the refusal ordering, and the attribution rule before any
promotion.

## Acceptance criteria

1. Independent review by the panel-contract owner, a docs curator, and a
   security reviewer accepts the namespace (`bitty.panel.overlay_show` /
   `overlay_hide`), the parameter grammar, and the closed plugin kind domain.
2. The gap statement is accepted as accurate: the Core writer exists, the
   plugin-facing verb does not, and `panel.overlay` remains unusable until a
   backend is specified and implemented.
3. Every borrowed bound is cited to its owner and unchanged; every proposed
   bound (OV-1 through OV-4) is labelled **proposed** with a rationale, and no
   accepted constant is restated as new.
4. The six-step check order is accepted, with the refusal guarantee that no step
   after the last allocation point can fail: a refusal allocates nothing.
5. Lifetime is accepted as stated: the raising plugin, generation release, panel
   disposal, and Core's own dismissal are the only paths, and a generation
   change retires every surface it owned.
6. The capability mapping is accepted: `panel.overlay` plus `ui.overlay`
   required, no input authority implied, and the bar and statusline claim rules
   unchanged.
7. The open points are either answered or explicitly deferred with an owner, and
   no accepted contract is weakened in the process.

## References

- [bitty-terminal/bitty#2476](https://github.com/bitty-terminal/bitty/issues/2476):
  residual 8, the request that a plugin-attributed overlay have a defined path.
- [bitty-terminal/bitty#2191](https://github.com/bitty-terminal/bitty/issues/2191):
  the band and overlay style parity work a raised surface must not regress.
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md): the
  closed slot set, the overlay gates, the presentation surface, and the stable v1
  error vocabulary.
- [bitty.ui Reference](../sdk/reference/ui.md): the current slot hosts, the
  ordered `ui.mount` checks, the per-generation bounds, and the typed denial
  table.
- [Plugin Platform RFC](plugin-platform-rfc.md): the capability grammar, grant
  lifecycle, presentation-not-truth, and the three-level queue budgets.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md): RC-1
  through RC-11 and the FS failure semantics that a refusal must satisfy.
- [Plugin UI Slot Inventory (Candidate)](../extensibility/plugin-ui-slot-inventory-candidate.md):
  per-slot admission, the `overlay` slot row, and the composition and claim
  rules a raise must compose with.
- [Plugin Drawing Capability (Candidate)](plugin-drawing-rfc-candidate.md): the
  sibling presentation proposal whose surface area bound (DR-6) OV-4 is
  comparable to, and whose denial style this document follows.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md):
  the accepted `4+1` envelope, `OverlayBusy`, and the rule that overlay geometry
  derives from the window logical area.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md):
  SCN-1 through SCN-5 and the block replacement rule.
- [Overlay Ownership and Composition Reconciliation](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/overlay-ownership-reconciliation.md):
  the three overlay owners, the tier and kind vocabularies, and the single modal
  authority this candidate composes with.
