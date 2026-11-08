---
title: Plugin Drawing Capability (Candidate)
description: Candidate contract for a bounded plugin drawing capability covering the instruction set, coordinate model, clipping, layers, animation, readback, resource bounds, damage, and the sandbox boundary
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 30
---

# Plugin Drawing Capability (Candidate)

> Status: **draft candidate** — not **Accepted**, not **Verified**, not
> **Compatible**, and not normative. This document does **not authorize
> implementation**, ships no behavior, and grants no compatibility promise.
> It exists because
> [bitty-terminal/bitty#2477](https://github.com/bitty-terminal/bitty/issues/2477)
> records that the drawing contract is undefined: the panel direction assumes
> custom content beyond the declarative v1 primitives, while
> `Panel::render(&mut Frame)`-style immediate-mode drawing is explicitly **not a
> plugin API today** and would hand plugins renderer, GPU, and window objects if
> it became one. This RFC answers that question with a data-only contract,
> separate from the panel and scene contracts it depends on.
>
> Every rule in this document is a **decision proposal**. "Proposes" means the
> text asks for review and acceptance; it never means the host implements it.
> Where a rule is not a proposal it cites an existing accepted contract by
> section, and no rule here may widen a value, ceiling, or gate owned by another
> document.

## Purpose and scope

In scope:

- the drawing model: what a plugin submits, in which coordinate space, with
  which instruction set, and how the host validates it;
- clipping, layer assignment, and z-order composition inside and outside a
  plugin contribution;
- whether plugins may animate, submit per-frame deltas, or request offscreen
  surfaces;
- readback: screenshot, pixel sampling, text extraction, and cross-panel
  capture, all deny-by-default behind an explicit capability;
- resource bounds for commands, bytes, surfaces, frame rate, memory, damage,
  and backpressure;
- the sandbox boundary the drawing path may not cross;
- determinism and replay, accessibility and semantic fallback, and the
  migration posture relative to the declarative v1 primitives.

Out of scope, and each owned elsewhere:

- the closed slot vocabulary, the v1 `SceneNode` subset, the `block_id` handle
  model, and `ui.rich`/`ui.overlay` gating (accepted,
  [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md));
- the capability identifier grammar, manifest schema, and deny-by-default
  grant lifecycle (accepted, [Plugin Platform RFC](plugin-platform-rfc.md));
- per-plugin instruction, memory, task, and queue ceilings (accepted,
  [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md));
- panel identity, lifecycle, focus routing, and the panel-level budgets
  (accepted, [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md));
- the declarative scene schema and its limits (accepted,
  [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md));
- which surfaces currently host a slot, and per-slot multiplicity and bounds
  (candidate, [Plugin UI Slot Inventory](../extensibility/plugin-ui-slot-inventory-candidate.md));
- the non-terminal panel content path (draft,
  [Panel Content Scene Path Decision](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-content-scene-path-decision.md)),
  which stays undecided and is named again under "Not in scope / depends on"
  below.

## Normative sources this specification must not weaken

- [Plugin Platform RFC](plugin-platform-rfc.md) (accepted): deny-by-default
  capability grammar with no wildcards, `ui.*` as a closed family,
  high-risk identifiers, the grant lifecycle including manifest-hash binding
  and revocation at the next dispatch boundary, the rule that presentation is
  never Terminal Truth, the rule that there are no hot-path events, and the
  three-level queue budgets with `DropOldest` as the accepted v1 default.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md)
  (accepted): RC-1 through RC-11 ceilings, FS-1 transactional denial through
  FS-9 no-silent-weakening, the per-VM isolation domain, and the adversarial
  cases that any drawing probe must extend rather than weaken.
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (accepted): the closed slot set, the v1 `SceneNode` restriction, opaque
  generation-owned handles, the L2 `ui.rich` gate, and the compatibility policy
  that makes additive minor versions the only growth path.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  (accepted): SCN-1 through SCN-5 scene limits, the replacement-by-`BlockId`
  rule, and the requirement that rich presentation never mutates terminal
  state.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  (accepted): panel identity, the single-owner hot path, the one-way geometry
  flow, and the panel capability family that a drawing grant must not subsume.
- [Security Overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md):
  invariants 2 (no ambient authority), 3 (presentation, never Terminal Truth),
  4 (no hot-path execution), 7 (bounded inputs), and 10 (`bitty --safe`).
- [Threat Model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md):
  T-06 (ambient authority), T-07 (callback and redraw storms), T-13 (alternate
  injection into grid state), and the untrusted-observation-data rule for T-10.

## Terminology

| Term                 | Meaning in this document                                                                                                     |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Draw list            | An ordered, bounded, immutable array of typed drawing commands submitted by a plugin and owned by the host after admission.  |
| Contribution         | One draw list admitted for one `(PluginId, generation, handle)` and painted into one host-owned region.                      |
| Content rect         | The host-assigned cell rectangle a contribution may paint into; its origin is `(0, 0)` in the plugin's coordinate space.     |
| Clip stack           | The ordered set of rectangles the host applies to a command; plugin-pushed clips may only narrow the content rect.           |
| Layer band           | One of the fixed ordered bands inside a contribution, painted in host-defined order.                                         |
| Surface              | A host-owned offscreen cell buffer a contribution may paint into before the host composites it.                              |
| Readback             | Any host-mediated return of pixels or text derived from a rendered surface rather than from the plugin's own submitted data. |
| Damage               | The set of cells a submission claims to have changed, validated by the host against the command bounding box.                |
| Deterministic replay | Re-rendering a recorded draw list into an identical frame with no plugin execution, no window, and no GPU.                   |

## Drawing model

### Contributions are data, not callbacks

The proposed model is a **retained bounded draw list**, not an immediate-mode
render hook. A plugin builds a value, the host validates it, and the host paints
it later on its own schedule. The rejected alternative is a
`Panel::render(&mut Frame)`-shaped callback in which plugin code runs inside the
renderer with a frame, device, or surface in hand; that shape is not a plugin
API today and this RFC does not propose making it one.

Consequences that the rest of this document depends on:

1. Plugin code never executes while a frame is being composed. Admission runs on
   the cold path, under the same callback accounting as any other host call
   (RC-1), and the renderer only ever reads host-owned validated data.
2. The VM boundary carries values, never handles: a draw list is a bounded
   array of plain typed values, matching the accepted service rule that results
   are values and not object handles into another VM.
3. The host may reject, coalesce, supersede, or drop a submission without
   calling back into the plugin, so a hostile submission cannot extend its own
   execution window by being slow to paint.

### Coordinate spaces

Three spaces were considered; the proposal admits exactly one in the first
slice.

| Space            | Definition                                                                                    | Disposition                                                                                                            |
| ---------------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Cell space       | Integer cells with `(0, 0)` at the top-left of the contribution's content rect.               | **Proposed** for the first slice.                                                                                      |
| Normalized units | Resolution-independent units (`0..=1000` on each axis) inside a host-sized offscreen surface. | Deferred; the host rasterizes at the display scale and never tells the plugin the scale, DPI, or physical pixel count. |
| Vector paths     | Move/line/quadratic/cubic/close with fill and stroke rules.                                   | Deferred; a path grammar is the largest single addition and is not needed to close the gap the issue names.            |

Rules:

1. **Cell space is anchored to the contribution, not to the window.** The plugin
   never learns or sets absolute coordinates, window geometry, slot thickness,
   or decoration geometry, consistent with the accepted rule that host layout
   owns placement.
2. **The plugin never learns the device scale.** Cell-to-pixel conversion,
   subpixel offsets, DPI, and font metrics stay host-owned. A command that
   depends on physical pixels is not expressible.
3. **Sub-cell positioning is not expressible in the first slice.** Raster
   precision arrives only with normalized units inside a surface, and only if
   [DRW-OQ-1](#open-questions) admits it.
4. **All geometry is validated integers.** Non-integer, negative,
   out-of-content-rect, and overflow-shaped values are rejected whole (FS-1),
   never clamped silently.

## Instruction set

The proposed first-slice instruction set is deliberately small: it is what a
panel needs to draw beyond text rows and columns, and nothing that requires the
host to trust a typed value.

| Command                  | Payload                                                         | Purpose                                                                            |
| ------------------------ | --------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `fill_rect`              | content-relative cell rect, theme style reference               | Backgrounds, bars, dividers, cell-aligned blocks.                                  |
| `text`                   | content-relative cell rect, one or more styled spans, alignment | Text runs using host-owned fonts, tokens, and shaping.                             |
| `image`                  | content-relative cell rect, admitted image identifier, fit mode | Placement of an image the plugin already obtained through the accepted image path. |
| `layer`                  | layer band name                                                 | Selects the band subsequent commands paint into.                                   |
| `clip_push` / `clip_pop` | content-relative cell rect                                      | Narrows the paint region for subsequent commands.                                  |
| `clear`                  | layer band name                                                 | Clears the contribution's own band to the host surface default.                    |

Explicitly **not** in the instruction set, in any slice proposed here:

- raw pixel or framebuffer writes, and any command carrying a pixel buffer;
- glyph identifiers, glyph injection, font selection, or shaping overrides;
- shaders, pipelines, blend modes beyond the host's fixed composition, blur,
  shadows, or filter graphs;
- cursor, selection, scrollback, mode, or grid commands — a draw list has no
  terminal target at all;
- drawing outside the contribution's content rect, into a sibling
  contribution, into host chrome, or into another plugin's surface;
- global z-order or stacking values, window placement, or decoration geometry;
- an unbounded command stream: a list is a finite value with a declared length,
  and there is no streaming or append-in-place form.

Text styling is expressed as references to host theme tokens, not as raw colors
or font features, so that contrast and appearance policy stay with the host
document that owns them. If a token reference is unknown, the command is
rejected rather than replaced by a fallback color.

## Clipping, layers, and z-order

1. **Clipping is intersect-only.** Every command is clipped to the intersection
   of the content rect, the current clip stack, and the band's own rect. A
   `clip_push` rectangle that is not fully inside the current clip is rejected
   with a typed error rather than silently narrowed, so a plugin cannot discover
   hidden geometry by probing.
2. **The clip stack is bounded** at DR-3 entries and is balanced: an unbalanced
   list is invalid, and `clip_pop` on an empty stack is invalid.
3. **Layers are four fixed bands**, painted in host-defined order:
   `background`, `content`, `foreground`, `annotation`. A contribution selects a
   band; it never selects an order, a z index, or a relative position against
   another plugin's contribution.
4. **Composition order is host-owned.** A contribution is one layer in the host
   surface's paint order, placed above the slot background and below host chrome
   and above-nothing overlays. The accepted overlay and modal authority stays
   Core-owned; a draw list can never paint above the Core capture surface.
5. **Paint order is a pure function of declared data.** For identical inputs the
   host produces identical output regardless of render timing, matching the slot
   inventory's composition rule.
6. **No plugin paints another plugin's cells.** Two contributions with
   overlapping content rects are ordered by the host surface's contributor
   order; the loser is clipped, never displaced, and never notified of the
   other's content.

## Frames, animation, and offscreen surfaces

### Frame ownership

The host owns the frame clock. A plugin **requests** a redraw and never produces
a frame:

1. `request_redraw` is a hint carrying no geometry and no payload. It is
   coalesced per contribution, bounded at DR-9, and may be ignored entirely when
   the surface is not visible or the host is over its frame budget.
2. A redraw hint never implies a new submission. A hint with no newer draw list
   is a no-op after the first coalesced occurrence.
3. A hidden, occluded, or suspended contribution receives no paint passes, so a
   plugin cannot convert visibility changes into CPU work.

### Animation

The proposal for the first slice is **no autonomous animation**: DR-7 is zero.
Concretely:

- a plugin cannot register a repeating timer for drawing purposes; the accepted
  task and timer caps (RC-4) already bound timers, and drawing adds no timer
  head;
- per-frame deltas driven by the plugin's own cadence are not expressible,
  because there is no per-frame callback to deliver them;
- a plugin cannot lower the host's frame interval, ask for a frame, or pin a
  surface visible.

If a later revision admits animation, the shape proposed here is a single
host-scheduled presentation tick per generation, strictly slower than the
display frame budget, charged to the plugin's callback budget under RC-1,
skipped when there is no damage, and marked by its own capability head
`ui.draw.animate` so that a draw grant alone never animates. Whether that tick
exists at all stays [DRW-OQ-2](#open-questions).

### Offscreen surfaces

Surfaces are the one place where the deferred raster space could live, and they
are the largest new allocation a drawing plugin could ask for. The proposal:

1. **At most DR-5 surfaces per `(PluginId, generation)`**, counted with the
   generation so disposal reclaims them (FS-5).
2. **A surface is host-allocated and host-owned.** It is addressed by an opaque,
   generation-scoped identifier; there is no pointer, no mapped buffer, no
   read-write view, and no way to pass a surface to another plugin.
3. **A surface has a fixed cell area (DR-6) and a host-chosen internal
   resolution.** The plugin supplies content-relative cell geometry only.
4. **A surface cannot be presented directly.** The host composites it into the
   contribution's content rect; a surface is never a window, a sub-surface, a
   native layer, or a compositor buffer.
5. **Surface memory is accounted, not free.** It counts inside the owning
   plugin's aggregate share under RC-3 and, where the pixels are reachable from
   the VM, inside RC-2. This RFC introduces no new heap ceiling row; if a
   reviewer decides surfaces need their own accounting row, that is a revision
   of the accepted RFC, not a local choice here.
6. **A refused surface request is total.** No handle, no reserved memory, no
   partial allocation (FS-1).

## Readback

Default posture: **deny**. There is no ambient readback, no readback implied by
`ui.rich`, `ui.draw`, `ui.overlay`, or `panel.*`, and no readback implied by a
service call from another plugin. The following table is the whole proposed
readback surface.

| Readback form            | Proposal                                                                                                                             | Grant                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| Own contribution text    | Return the plugin's own declared semantic tree as bounded values. No pixels involved.                                                | `ui.readback.self`                 |
| Own contribution pixels  | Return sampled values from the plugin's own surface, before host composition, as host-encoded bounded values.                        | `ui.readback.self`                 |
| Screenshot of the window | Not a plugin capability in this RFC. Screenshots stay a user and Core action.                                                        | None proposed                      |
| Cross-panel capture      | Permanently denied. No identifier is proposed, so no grant can exist; a future grant needs its own threat-model entry first.         | None proposed                      |
| Terminal text or pixels  | Never routed through drawing readback. Terminal content stays with `terminal.semantic-read`, and raw cells with `terminal.raw-read`. | Existing accepted identifiers only |

Rules:

1. **Own-content-only is a property, not a policy.** Readback is scoped to the
   contribution's own surface staged before host composition, so a sampled value
   can never contain another plugin's content, terminal cells, host chrome, or
   the cursor.
2. **Bounded per call and per minute** (DR-11, DR-12). The host returns decoded
   values, never a buffer handle, an image object, or an encoder callback.
3. **Text extraction reads declared structure, not rendered glyphs.** If the
   plugin did not declare semantics, there is nothing to extract; the host does
   not guess text back out of its own rasterizer.
4. **Readback can never compose into authority.** A sampled value is
   observation data; it grants no input, clipboard, filesystem, or terminal
   write authority, and the accepted rule that authority follows the requesting
   plugin is unchanged.
5. **A denied readback is a typed denial, not an empty result.** The plugin
   cannot distinguish "no content" from "no permission" except through the
   error class, which keeps probing unprofitable.

## Capability names and denial taxonomy

The capability identifier grammar is accepted and closed; this RFC proposes
additions inside the existing `ui` family and nothing outside it.

| Proposed identifier | Effect                                                                        | Risk posture                                                                                           |
| ------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `ui.draw`           | Submit bounded draw lists into a drawing-capable slot.                        | Presentation-only; never bundled with terminal write, which does not exist in v1.                      |
| `ui.draw.animate`   | Opt into the host-scheduled presentation tick, if animation is ever admitted. | Not granted by `ui.draw`; separate consent head, following the `ui.overlay.focus` precedent.           |
| `ui.readback.self`  | Read back the plugin's own contribution text and pixels.                      | Needs distinct consent wording and its own diagnostic class; never implied by any draw or mount grant. |

Proposed unified diff classes for denials, matching the accepted pattern of a
typed error plus a class:

| Code                 | Class        | Meaning                                                                             |
| -------------------- | ------------ | ----------------------------------------------------------------------------------- |
| `E_DRAW_UNAVAILABLE` | `capability` | The target slot has no drawing host in this build.                                  |
| `E_DRAW_CAPABILITY`  | `capability` | The required grant is absent, revoked, or not declared in the manifest.             |
| `E_DRAW_LIMIT`       | `budget`     | A DR ceiling, the content rect, or a scene limit was exceeded.                      |
| `E_DRAW_INVALID`     | `validation` | Malformed value: bad geometry, unbalance clip, unknown token, unknown command.      |
| `E_DRAW_DAMAGE`      | `validation` | Declared damage does not cover the command bounding box.                            |
| `E_DRAW_STALE`       | `lifecycle`  | The handle or generation is disposed or superseded.                                 |
| `E_DRAW_BUSY`        | `budget`     | A pending submission occupies the single slot (DR-8); retry is the plugin's choice. |
| `E_READBACK_DENIED`  | `capability` | Readback was requested without the grant, or for a scope that has no identifier.    |

Denial semantics inherit the accepted failure rules: transactional (FS-1), typed,
attributed (FS-4), and never silent (§ FS-9). A denial is not an empty frame.

## Resource bounds

The drawing path introduces no new global budget family. It nests beneath the
accepted ceilings, cited by identifier rather than restated:

- callback CPU and wall time for admission: **RC-1**;
- per-VM accounted memory: **RC-2**; aggregate plugin memory: **RC-3**;
- tasks and timers, which drawing adds no head to: **RC-4**;
- queue depth and drop accounting, whose authoritative drop-policy statement is
  the accepted `DropOldest` default for UI observation systems: **RC-5**;
- persistent state, if a plugin caches a surface description: **RC-11**.

Drawing-specific dimensions are proposed below. These values are **candidates**,
not accepted ceilings; on acceptance they become owned values that change only
by RFC revision, and every implementation test must parameterize on them rather
than hard-code them.

| ID    | Dimension                            | Candidate value                            | Validation point | Failure             |
| ----- | ------------------------------------ | ------------------------------------------ | ---------------- | ------------------- |
| DR-1  | Commands per draw list               | 1024                                       | admission        | `E_DRAW_LIMIT`      |
| DR-2  | Serialized draw list bytes           | 64 KiB                                     | admission        | `E_DRAW_LIMIT`      |
| DR-3  | Clip stack depth                     | 8                                          | admission        | `E_DRAW_LIMIT`      |
| DR-4  | Layer bands per contribution         | 4                                          | admission        | `E_DRAW_INVALID`    |
| DR-5  | Offscreen surfaces per generation    | 1                                          | admission        | `E_DRAW_LIMIT`      |
| DR-6  | Offscreen surface area               | 4096 cells, no axis above 256 cells        | admission        | `E_DRAW_LIMIT`      |
| DR-7  | Animation frames per second          | 0 in the first slice                       | policy           | `E_DRAW_CAPABILITY` |
| DR-8  | Pending draw lists per contribution  | 1, newest supersedes                       | admission        | `E_DRAW_BUSY`       |
| DR-9  | Redraw hints per second              | 10, coalesced                              | admission        | silently coalesced  |
| DR-10 | Semantic nodes per draw list         | 256, and never above the SCN-1 node budget | admission        | `E_DRAW_LIMIT`      |
| DR-11 | Readback region and payload per call | 64 × 64 cells, 16 KiB returned             | call             | `E_READBACK_DENIED` |
| DR-12 | Readback calls per minute            | 60                                         | call             | `E_READBACK_DENIED` |

Two invariant rules hold regardless of the numbers:

- **Never above the owner.** No DR value may raise an RC ceiling, a SCN limit, a
  panel budget, or a slot bound; the effective bound is the minimum of every
  applicable one, and a lower layer can only tighten.
- **Attributed or absent.** Every ceiling is enforced per `(PluginId,
generation, handle)` and emits observable accounting. Enforcement that cannot
  run means the capability is refused, not that it runs unbounded (FS-7).

## Damage tracking and backpressure

1. **The host computes the true extent.** It derives the bounding box of the
   submitted commands after clipping and compares it to the declared damage.
   Under-claimed damage is rejected (`E_DRAW_DAMAGE`) rather than painted stale.
2. **Over-claimed damage is clipped, not trusted.** Damage outside the content
   rect is discarded, and requests that would invalidate the whole window are
   reduced to the contribution's own rect. A plugin cannot force a full-surface
   repaint.
3. **Damage composes upward, never sideways.** Contribution damage unions into
   the host surface's damage, which unions into the frame's damage. The terminal
   grid path and the panel content path stay separate; a drawing contribution
   never adds damage to a terminal view and never re-enters the parser or grid
   commit path.
4. **Damage is a set of cells, not a counter of requests.** Repeated identical
   damage coalesces; the dominant cost of a hostile submission is one bounded
   validation pass, not per-request painting.
5. **Presentation is not truth.** Damage composition changes what the compositor
   repaints. It never mutates grid cells, cursor position, modes, scrollback,
   selection, or reply buffers, and it has no path to do so.
6. **Backpressure is drop-newest-submission, never a queue.** One pending draw
   list exists per contribution (DR-8). A newer list supersedes the pending one;
   the older one is discarded and counted. Submissions are never queued, never
   merged, and never block the plugin's VM past its host-call budget.
7. **Supersession is observable.** Superseded and coalesced counts are
   attributed per plugin and reported through the existing plugin diagnostics
   surface, so a redraw storm is diagnosable instead of silent.

## Sandbox boundary

A plugin never receives, holds, or names any of the following, in the VM or in a
marshalled value:

- the renderer, a frame, a device, a queue, a texture, a shader, a pipeline, or
  any GPU object or handle;
- an OS window, a native layer, a compositor buffer, a sub-surface, or a
  presentation id;
- a surface pointer, mapped memory, or a read-write view of any surface;
- a font, glyph, or shaping object, or any value that exposes physical pixel
  metrics;
- a native code path: no `.so`/`.dll`/`.dylib`, no FFI, no shared-library
  loading, consistent with the accepted prohibition on native in-process
  payloads;
- an unbounded or streaming command form, and no callable object that paints.

And the drawing path never places a plugin in these paths:

- the parser, the grid commit, the damage-to-snapshot path, or the encoder;
- the input, keymap, or focus hot path — a draw list carries no input
  authority, and no command in it can request focus, capture keys, or consume
  an event;
- the frame composition loop, where only host-owned validated data is read.

`Panel::render(&mut Frame)` is **not a plugin API today**, and this RFC does not
propose one. An immediate-mode hook is incompatible with rules 1 and 3 above: it
would require plugin execution inside the render pass with live renderer state
in scope, which is exactly the boundary the accepted isolation model exists to
hold. A future proposal that wants one must first revise the isolation contract,
not this one.

## Determinism and replay

1. **Same input, same output.** A draw list, a content rect, and a theme token
   set deterministically produce the same frame; the host may cache and diff by
   content hash.
2. **Replay needs no plugin, no window, and no GPU.** A recorded draw list can be
   replayed headlessly into a comparable frame, in the same style as the existing
   headless measurement harnesses. This is the test mechanism for bounds,
   denial, and determinism, and it is why the retained-list model is preferred
   over a callback.
3. **No ambient inputs enter painting.** Time, random numbers, locale, and
   device state are not readable from the paint path; if a plugin wants variety
   it must put the value in the draw list, where it is recorded and replayable.
4. **Cross-platform equality is per-host, not promised.** Replay equality is
   promised for one host build and one token set; matching pixel output across
   font stacks is not claimed.

## Accessibility and semantic fallback

Custom drawing is allowed to be unreadable to a screen reader only if the host
can say so honestly.

1. **Semantics are part of the submission.** Every draw list carries a bounded
   semantic tree (DR-10) with node kinds the host already understands, and text
   extraction reads that tree, not the rasterizer.
2. **Missing semantics is a diagnostic, not a silent gap.** A contribution with
   painting but no semantics is admitted, is reported as semantically empty to
   the claimant, and is represented in the accessibility tree by a host-authored
   placeholder naming the plugin. It is never silently omitted and never
   described with guessed text.
3. **Semantics cannot contradict paint silently.** Where the semantic tree and
   the painted content disagree, the semantic tree is authoritative for
   assistive output and the disagreement is reported; the host does not
   synthesize a second description from pixels.
4. **No decoration hides another surface's text.** A drawing contribution is
   confined to its own content rect, so it cannot occlude terminal text from the
   accessibility tree, matching the accepted rule that decorations must not hide
   grid text without an explicit replacement claim.
5. **Reduced motion is host policy.** With DR-7 at zero the question is
   dormant; if animation is admitted, reduced-motion preference suppresses it
   without plugin cooperation and without changing what is painted at rest.

## Relationship to the declarative surface

This contract is separate from the declarative v1 primitives by design; the
issue asks for a drawing contract that is not coupled to them. The two meet in
exactly one place: slot admission.

| Question                          | Declarative path (accepted)                           | Drawing path (this candidate)                                 |
| --------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| What is submitted                 | A `SceneNode` subtree in `Text`/`Row`/`Column`/`List` | A bounded draw list plus a semantic tree                      |
| How it is bounded                 | SCN-1 through SCN-5 per block                         | DR-1 through DR-12 per contribution                           |
| What gates it                     | `ui.rich`, plus `ui.overlay` for the overlay slot     | A separate `ui.draw` grant; never implied by `ui.rich`        |
| What renders it                   | Host scene composition                                | Host draw-list validation and rasterization                   |
| What changes for existing plugins | Nothing                                               | Nothing; no grant exists by default and no slot changes shape |

Migration posture, as authoring guidance rather than contract:

1. Declarative primitives stay the default and the cheapest path. Drawing is
   opt-in per contribution and per grant.
2. There is no automatic translation in either direction. A plugin that wants
   drawing submits a draw list; it does not decorate, wrap, or embed a scene
   subtree, and a scene node does not gain a paint payload.
3. A plugin that only needs alignment, tables, or styled text should keep the
   declarative path; the drawing path is for content the node subset cannot
   express, and its accessibility cost is real.
4. A drawing contribution counts as one contribution for slot multiplicity and
   block accounting, exactly like a declarative one, so it cannot be used to
   exceed a slot bound.

## Not in scope / depends on

This RFC depends on decisions it does not make. Nothing below is settled by
this document, and no implementation may treat a drawing decision as settling
one of them.

1. **Panel lifecycle ownership.** Who creates, destroys, suspends, and revives a
   panel, and how a panel's generation relates to a plugin generation, stays
   with the accepted panel contract and its open questions. This RFC only
   assumes that whatever owns the panel also owns the content rect a
   contribution is clipped to, and that a disposed generation's contributions
   are reclaimed.
2. **The scene path.** Whether panel content reaches the renderer as a
   declarative scene, as a per-leaf grid snapshot, or through some successor
   remains a draft decision, and the consumption contract is not accepted. This
   RFC therefore specifies what a plugin submits and what the host validates,
   and deliberately says nothing about the internal transport that carries the
   result to the renderer.
3. **Which slots can host drawing.** The slot inventory is a candidate and the
   current host set is narrower than the accepted vocabulary; `ui.draw` is
   proposed as an admission gate, not as a way to make an unhosted slot
   available.
4. **Presentation-mode and placement rules.** Panel placement, floats, tiers,
   and workspace persistence are owned by the panel and compositor contracts.
   A draw list has no placement opinion.
5. **Image decode, storage, and budgets.** An `image` command references an
   already-admitted image; decode limits, stores, and format policy stay with
   the accepted rich-presentation contract.
6. **Theme tokens and contrast.** Style references resolve against host tokens
   owned by the appearance and theme contracts; this RFC defines no color
   values and no contrast rule.

## Open points

Non-question items still to be written down before review can close:

- the final spelling of the error codes above, which should match the host's
  existing typed-error conventions;
- whether a drawing contribution consumes one block of the accepted per-terminal
  block budget, and how that is counted when the contribution also owns a
  surface;
- the exact diagnostic channel that reports superseded and coalesced
  submissions.

### Open questions

1. **DRW-OQ-1** — Do raster drawing in normalized units and vector paths ever
   enter the contract, and in which slice? The first slice can close the gap
   with cell space alone; vector work is the largest single addition and would
   need its own accessibility story.
2. **DRW-OQ-2** — Is animation admitted at all? If yes, at what host-scheduled
   frequency, with what damage bound, and does the accepted no-periodic-timer
   posture for presentation survive a plugin-driven tick in any form?
3. **DRW-OQ-3** — Should offscreen surfaces beyond the single self-surface
   exist, and should surface memory get its own accounting row instead of
   nesting under RC-3?
4. **DRW-OQ-4** — Should `ui.readback.self` include pixels at all, or is
   text-and-semantics readback sufficient? Pixels add an exfiltration surface
   for content the plugin did not author, if the staging boundary is ever
   wrong.
5. **DRW-OQ-5** — Is a user-visible screenshot permanently outside the plugin
   capability space, or is it a future high-risk capability with its own
   threat-model entry and consent design?
6. **DRW-OQ-6** — What is the aggregate bound per drawing-capable slot, and how
   is it attributed across plugins? This depends on the slot inventory's own
   open aggregate bound and cannot be fixed here.
7. **DRW-OQ-7** — Should `ui.draw` be a distinct head, or an additive scope
   under the accepted `ui.rich`? A distinct head keeps the consent story
   honest; a scope keeps the identifier count down.
8. **DRW-OQ-8** — Are alpha and blend modes ever admitted, and if so, does
   compositing against host-owned content require a separate consent head?

## Security review

This proposal creates no ambient authority and relaxes nothing it cites.

| Element                                               | Gate it serves                                                     | Threat/risk        |
| ----------------------------------------------------- | ------------------------------------------------------------------ | ------------------ |
| Draw lists are data; no callback in the render pass   | Invariant 4 (no hot-path execution); accepted no-hot-path rule     | T-07               |
| No GPU, window, frame, surface pointer, native code   | Invariant 2 (no ambient authority); native payload prohibition     | T-06, R-017        |
| Deny-by-default capability heads; no implied readback | Deny-by-default grammar, no wildcards, distinct consent            | T-06, T-13         |
| Readback scoped to own pre-composition surface        | Confused-deputy and cross-tenant data exposure                     | T-06, T-10         |
| Cross-panel capture has no identifier                 | No grant can exist, so no escalation path exists                   | T-06, T-13         |
| Damage confined to the content rect                   | Availability; no full-window invalidation; no terminal damage path | T-07, T-13         |
| Single pending submission, drop-newest                | Backpressure isolation; no unbounded queue                         | T-07               |
| Deterministic replay and headless rendering           | Testability of bounds, denial, and determinism                     | verification gates |
| Schema and command validation before allocation       | Untrusted input handling; fuzzable surface                         | T-06, T-12         |

Residual risks a reviewer must weigh:

- **Readback is the sharp edge.** Even own-content-only readback is a channel
  whose safety rests entirely on the staging boundary being before composition.
  If that boundary is ever implemented after composition, the capability becomes
  a cross-content capture capability; the contract must say so explicitly.
- **Drawing raises the accessibility floor.** A cell-space painter is easier to
  get wrong for assistive output than the declarative node set, which is why
  semantics are mandatory and their absence is reported rather than ignored.
- **A single surface is still a surface.** Bounded memory is not free memory;
  DR-5 and DR-6 must be enforced at admission, not hoped for at paint.

No `P0` criterion is modified by this document. A security reviewer is required
for the capability heads, the readback contract, and the error taxonomy before
any promotion.

## Verification plan

Acceptance of an implementation later requires, at minimum:

1. **Bounds:** each DR ceiling is enforced at admission at its value, at its
   floor, and at a policy maximum, with typed denials and no partial state
   (FS-1); a draw list one command or one byte over budget is rejected whole.
2. **Denial:** every row of the denial taxonomy is reachable by a test, every
   capability head denies by default, and `ui.rich` alone never admits a draw
   list.
3. **Readback:** a plugin without `ui.readback.self` cannot observe pixels or
   text; a granted plugin cannot observe another contribution, terminal content,
   chrome, or the cursor; a cross-panel request has no identifier to name.
4. **Sandbox:** an ambient-authority probe extended with frame, device, window,
   surface-pointer, and native-load attempts is denied on every entry, mirroring
   the accepted adversarial ambient-authority case at the drawing boundary.
5. **Damage:** under-claimed damage is rejected, over-claimed damage is clipped
   and cannot invalidate the window, and composition never adds damage to a
   terminal view.
6. **Backpressure:** a submission storm leaves exactly one pending list, counts
   every supersession, and never blocks the plugin's VM past its host-call
   budget.
7. **Determinism:** replaying a recorded draw list produces an identical frame
   with no plugin execution and no window, and repeated replay is stable.
8. **Accessibility:** a contribution with semantics yields the declared tree; a
   contribution without semantics yields the named host placeholder and a
   diagnostic, never a guessed description.
9. **Safe mode:** `bitty --safe` admits no drawing contribution and still boots
   (invariant 10).

## Alternatives considered

| Alternative                                        | Trade-off                                                                                                         | Disposition                                                                        |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Immediate-mode `render(&mut Frame)` callback       | Most familiar to UI authors; expressive. Puts plugin code in the render pass with renderer state in scope.        | Rejected: violates the sandbox boundary this RFC exists to hold.                   |
| Raw pixel buffer submission                        | Maximum freedom, simple to specify. An unbounded untrusted buffer in the paint path, with no cheap validation.    | Rejected: no bounds story and no accessibility story.                              |
| Vector path grammar in the first slice             | Resolution independent and expressive. Largest new surface, plus hit-testing and a11y questions nobody has asked. | Deferred: not needed to close the named gap.                                       |
| Reuse `ui.rich` as the drawing gate                | One identifier, fewer prompts. A declarative mount grant would silently become a paint grant.                     | Rejected: distinct authority needs distinct consent.                               |
| Allow animation with a plugin-driven timer         | Motion is desirable for some panels. Turns a presentation feature into an event-rate feature under RC-4.          | Rejected for the first slice; host-scheduled tick proposed only as a future shape. |
| Allow cross-panel capture behind a high-risk grant | Useful for reporter and accessibility tooling. Creates a cross-content observation channel with no owner.         | Rejected: no identifier proposed; needs its own threat-model entry first.          |
| Stream commands rather than submit a list          | Lower latency for large content. Unbounded stream, unbounded validation cost, non-replayable.                     | Rejected: bounded list only.                                                       |

## Affected contracts

| Contract                                                                                                                                                         | Effect if this candidate is accepted                                                                  |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| [Plugin Platform RFC](plugin-platform-rfc.md) (accepted)                                                                                                         | Gains three `ui.*` identifiers and the drawing denial classes; no grammar change.                     |
| [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (accepted)                                                                                    | Gains a drawing probe in the adversarial set and, if DRW-OQ-3 resolves that way, one accounting note. |
| [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted)                                                                              | Gains an additive drawing entry point in a minor version; the `SceneNode` subset is unchanged.        |
| [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md) (accepted)                      | Unchanged; SCN limits remain the declarative-side bounds.                                             |
| [Plugin UI Slot Inventory](../extensibility/plugin-ui-slot-inventory-candidate.md) (candidate)                                                                   | Gains a drawing-capable admission question per slot.                                                  |
| [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md) (accepted)                              | Unchanged; drawing cannot widen a panel budget or claim lifecycle.                                    |
| [Panel Content Scene Path Decision](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-content-scene-path-decision.md) (draft) | Unchanged; the scene path stays undecided and is a stated dependency.                                 |
| [SDK bitty.ui reference](../sdk/reference/ui.md) (draft reference)                                                                                               | Gains the drawing surface beside the declarative one once behavior exists.                            |

No new repository, crate, workflow, or global budget family is introduced.

## Acceptance criteria

1. Independent review by the category owner, a docs curator, and a security
   reviewer accepts the instruction set, the capability heads, the denial
   taxonomy, and the readback posture.
2. The sandbox boundary is accepted as stated: no frame, GPU, window, surface
   pointer, or native handle reaches a plugin, and no plugin code runs in the
   render, input, or parser path.
3. Every bound is stated, attributed, and no wider than the RC, SCN, panel, or
   slot value it nests beneath.
4. Tests for bounds, denial, determinism, damage, and backpressure are named and
   scoped, and the reference implementation scope is recorded.
5. The open questions above are either answered or explicitly deferred with an
   owner, and no accepted contract is weakened in the process.

## P0 Review Sign-off

Not applicable yet: this document is a candidate. It records a required security
review (trust boundary, capability heads, resource limits, and readback) but
carries no sign-off, and no P0 criterion is modified until the review above
completes and this document's status changes through the documented lifecycle.

## References

- [bitty-terminal/bitty#2477](https://github.com/bitty-terminal/bitty/issues/2477):
  the request that a drawing contract be defined separately from the panel
  contract.
- [Plugin Platform RFC](plugin-platform-rfc.md): capability grammar, grant
  lifecycle, presentation-not-truth, no hot-path events, queue budgets.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md): RC
  ceilings, failure semantics, adversarial test specification.
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md):
  closed slot set, `SceneNode` subset, handle and compatibility model.
- [Plugin UI Slot Inventory (Candidate)](../extensibility/plugin-ui-slot-inventory-candidate.md):
  per-slot admission, multiplicity, and conflict rules.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md):
  scene limits and the declarative contribution model.
- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  and [Panel Content Scene Path Decision](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-content-scene-path-decision.md):
  panel identity, lifecycle, and the undecided scene path.
- [SDK bitty.ui reference](../sdk/reference/ui.md): current declarative UI
  behavior and its draft status.
