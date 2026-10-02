---
title: Search and Copy-Mode Policy and Public API Requirements
description: Draft plugin-side policy and public-API requirements for the search and copy-mode plugins over the Core-owned bounded search, selection, clipboard, and input-capture mechanisms
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 28
---

# Search and Copy-Mode Policy and Public API Requirements

## Document status

> Status: **draft** — not **Accepted**, not **Verified**, not **Compatible**,
> and not normative. This document records the plugin-side policy and
> public-API requirements for the `search` and `copy-mode` plugins whose
> mechanisms stay in Core. It authorizes no shipped, stable, or
> compatibility-guaranteed behavior, weakens no accepted source it cites, and
> makes no implementation claim. Exact host and SDK spellings are delegated to
> `W-139`; package experience and result presentation are owned by `CTX-0003`;
> the terminal-side mechanism contract is owned by `W-135`. Names and shapes
> repeated here are requirements and direction, not contract.

The plugin-facing inputs this document depends on are owner-pending or in
flight: the terminal-side [Search, Selection, and Snapshots
Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/search-selection-contract.md)
(`W-135`), the SDK surface (`W-139`), the focusable-overlay and transient
input-capture host API (`W-01`, Issue
[#396](https://github.com/bitty-terminal/bitty-docs/issues/396), under `OQ-056`),
and the search/copy-mode package (`CTX-0003`). `OQ-074` (scrollback search UX)
and `OQ-075` (keyboard-selection and copy mode) remain **Open**. This page
references those owners and decides none of their content.

- Owning task: `W-138` (bitty-plugins-docs), CarryCtx `CTX-0073`, Issue
  [bitty-plugins-docs#130](https://github.com/bitty-terminal/bitty-plugins-docs/issues/130).
- Terminal-side mechanism: `W-135`, CarryCtx `CTX-0091`, Issue
  [bitty-terminal-docs#168](https://github.com/bitty-terminal/bitty-terminal-docs/issues/168).
- SDK surface: `W-139`, CarryCtx `CTX-0066`.
- Package experience: search/copy-mode package `CTX-0003`.

## Purpose and scope

Core keeps the search mechanism, selection semantics, and clipboard permission
that the terminal-side contract defines. This document fixes the **plugin-side
policy and the public-API requirements** the search and copy-mode plugins must
satisfy when they consume that mechanism, so the package, the SDK binding, and
the Core integration can be built against one reviewed plugin-facing input
without re-deciding where the trust boundary sits.

In scope (all **Draft** unless citing an accepted source):

- S: the search plugin policy — input box UX, result presentation, result
  navigation, keybindings, result identity handling, and owner-frame-only
  highlighting — over the Core-owned bounded search API;
- C: the copy-mode plugin policy — modal keybindings, cursor movement, and
  selection interaction — over Core-owned selection semantics;
- I: the input-capture dependency (`W-01` / Issue #396 / `OQ-056`) and the
  Composer and Beacon capture rules it composes with, referenced but not
  decided;
- A: the high-level public-API requirements for snapshot and viewport reads,
  navigation, selection get and set, clipboard writes, and the no-hot-path
  and no-Terminal-Truth rules, with exact spellings delegated to `W-139`;
- K: the capability declarations each plugin requests, deny-by-default, with
  no widening and no private first-party bypass;
- P: the standard plugin page set for the search and copy-mode packages as
  real-content guidance;
- an explicit statement of what Core retains, a security review, a verification
  plan with negative-path evidence, alternatives, affected contracts, open
  points, and acceptance criteria.

Out of scope and owned elsewhere (pointers, not content):

- the bounded snapshot surface, stable identity, viewport navigation, result
  coalescing, selection semantics, clipboard permission, bounded search, and
  the retained mechanisms themselves (`W-135`; that page is authoritative);
- the exact SDK function names, argument tables, result schemas, and error
  identifiers (`W-139`, `CTX-0066`); this page states high-level requirements
  only and marks search/copy-mode-facing operations provisional;
- the package delivery shape (Lua plugin or Rust-level extension), manifest
  compatibility declaration, keybinding namespace, case and regex scope, and
  result presentation (`CTX-0003`, `W-138`);
- the `W-01` host primitive spellings, capture event payloads, focus-order
  rules, and timeout values; this page references `W-01` and decides none of
  them;
- the Core integration that wires the plugins to the host (`W-143` `CTX-0936`
  and `W-144` `CTX-0937`);
- IME capture, pointer capture, the Leader or chord namespace, and mouse-mode
  precedence, which stay with their owning input contracts and `OQ-075`;
- the renderer's scene composition and present path, owned by the accepted
  terminal-side Rich Presentation RFC and scene contracts.

## Normative sources this specification must not weaken

This document must be read together with, and must not weaken:

- The terminal-side [Search, Selection, and Snapshots
  Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/search-selection-contract.md)
  (`W-135`): the bounded per-view snapshot, stable identity and generation
  fencing, viewport navigation, result coalescing, selection semantics,
  clipboard permission, bounded search, and owner-frame highlighting rules
  this page consumes.
- The bitty-docs security corpus:
  [security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md),
  [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md),
  [risk register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md),
  and
  [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md).
  The binding controls include suspicious paste inspection (`P0-AC-008`),
  deny-by-default local files (`P0-AC-005`), hyperlink and process launch
  without shell interpolation (`P0-AC-009`), capability-checked host APIs and
  official-plugin parity (`P0-AC-012`), per-plugin resource budgets
  (`P0-AC-014`), exclusion from the input, parser, and render hot paths
  (`P0-AC-015`), Core-owned Terminal Truth (`P0-AC-016`), safe mode
  (`P0-AC-019`), trace minimization (`P0-AC-026`), and the panel lease write
  gate (`P0-AC-039`).
- [ADR 0015 - Small-Core Extraction
  Boundaries](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0015-small-core-extraction-boundaries.md):
  Terminal Truth and permission are retained, there is no private first-party
  bypass, no raw PTY, GPU, or window handle, and no input hot-path callback.
- [ADR 0016 - Execution, Graphics, Accessibility, Storage, and Platform-Service
  Boundaries](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md):
  search and selection are not a `W-130` boundary; `W-135` and `W-138` stay
  gated on `W-01` and `OQ-074`/`OQ-075`.
- The accepted [Plugin Platform
  RFC](plugin-platform-rfc.md): the closed capability identifier grammar and
  families, deny-by-default grants, official-plugin parity, the manifest,
  consent, revocation, and update-diff model, the Plugin API v1 surface, and
  the no-hot-path invariant.
- The accepted [Plugin Manifest and Capability Grammar
  Authority](manifest-capability-authority.md): the canonical capability
  spellings and the closed v1 family set; a new family cannot be invented by
  a document or a plugin.
- The accepted [Plugin API v1 Lua Surface
  RFC](../sdk/plugin-api-v1-lua-surface-rfc.md): the frozen v1 function
  surface, `bitty.terminal.snapshot`, the `terminal.semantic-read` gate, the
  `SNAPSHOT_MAX_BYTES` bound, the key-binding suggestion contract, and the
  explicit statement that v1 exposes no write path to grid, cursor, modes, or
  scrollback.
- The accepted [Isolation and Resource
  RFC](../runtime/isolation-resource-rfc.md): per-plugin budgets and the
  cross-cutting resource caps any read, search, or highlight work consumes.
- The terminal-side [Input and Pointer
  Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/input-pointer-rfc.md)
  and [Composer Architecture and Host
  API](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md),
  plus the [Beacon Core Mechanism
  Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md):
  the selection model, the `CLIPBOARD_MAX_BYTES=8192` bound, the `W-01`
  focusable-overlay and transient input-capture consumer shape, and the
  rule that transient capture is Core-owned, generation-fenced, revocable, and
  never a plugin callback on the input hot path.
- The terminal-side [Performance Budget
  RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/performance-budget-rfc.md):
  `PB-4` input latency and the invariant that plugins do not enter the input
  hot path.
- The terminal-side [Compatibility Milestone
  RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/compatibility-milestone-rfc.md):
  the mouse modes, focus reporting, alternate scroll, and bracketed paste the
  search and copy interactions must not change.

## Terminology

| Term                     | Meaning in this document                                                                                                                                          |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Core                     | The always-available terminal mechanism that works with zero plugins and in `bitty --safe`; it owns Terminal Truth, search, selection semantics, and permission.  |
| Search plugin            | The extension that owns search UX policy over the Core-owned bounded search API; it never owns the search mechanism.                                              |
| Copy-mode plugin         | The extension that owns modal copy interaction policy over Core-owned selection semantics; it never mutates Terminal Truth.                                       |
| Policy                   | Optional behavior and presentation the extension owns: UX, keybindings, result layout, navigation gestures, and styling.                                          |
| Bounded search API       | The Core-owned, deterministic, finite search over Terminal Truth that returns bounded, generation-stamped results over one owning view.                           |
| Snapshot                 | A bounded, read-only, cold-path projection of one owning view's Terminal Truth, copied out for an extension; never a mutable handle.                              |
| Stable identity          | A reference to content that survives scroll, resize, reflow, and pruning by pairing an owning view with a stable line id when available, plus a generation.       |
| Generation               | A monotonic stamp attached to a snapshot, result, selection, or capture handle so a stale reference is rejected by Core, not trusted from the caller.             |
| Selection                | A presentation-layer range over grid cells; not Terminal Truth.                                                                                                   |
| Selection semantics      | Core's anchor and focus, word, line, and rectangular block expansion, normalization, wide-char snapping, reclamp, and pruning truncation rules.                   |
| Clipboard permission     | The Core capability and consent gate around clipboard read and write, distinct from a direct trusted user copy gesture; bounded by `CLIPBOARD_MAX_BYTES=8192`.    |
| Owner-frame highlighting | A presentation-only projection that paints results over the owning view's grid, clipped to that view's column window and content frame; never global coordinates. |
| Input capture            | The capability-gated Core host mechanism that takes exclusive keyboard and overlay focus for one UI interaction, transiently and revocably.                       |
| Typed outcome            | A structured result (success, denied, stale, timeout, unavailable) that names the missing capability or violated rule on denial, never a bare boolean.            |
| Capability               | A named, narrowly scoped authority from the closed v1 family grammar; deny-by-default, grant-bound to the manifest hash.                                          |
| Page set                 | The standard four-page per-plugin documentation partition: `README.md`, `design.md`, `schemas.md`, and `evidence.md`.                                             |
| No-plugin mode           | The startup and runtime state in which the search/copy-mode plugin is absent, disabled, failed, or incompatible; the retained Core behavior applies.              |
| Safe mode                | `bitty --safe`, which starts with zero third-party plugins and no optional policy; Core search and selection remain usable.                                       |

## Search plugin policy

The search plugin owns the search **experience**. Core owns the search
**mechanism**. Every requirement below is policy over the plugin-facing API the
`W-135` contract defines and the `W-139` surface will bind; the plugin never
implements matching, ordering, case folding, or bounds.

### Input box UX

- The plugin presents one bounded search input surface, typically a focusable
  overlay opened by a keybinding suggestion. The field text, placeholder,
  prompt, and any match counter are bounded display text, never terminal
  content and never markup.
- Focus acquisition uses the `W-01` focusable-overlay and transient
  input-capture host API; while the input surface holds capture, captured keys
  produce no PTY bytes and do not leak to the terminal.
- The plugin chooses incremental or submit-driven querying as policy, but it
  must query through the Core-owned bounded search API on a safe boundary; it
  may not install a per-keystroke callback on the input hot path.
- The plugin may expose case, whole-word, and regex scope toggles as UX, but
  the accepted headless search folds ASCII only when case-insensitive, and
  regex versus literal and Unicode case-folding scope stay with `OQ-074`; the
  plugin must not claim or assume an unaccepted matching mode, and a mode the
  host does not implement stays disabled or absent.
- The plugin respects the finite pattern bound: an over-limit pattern is
  truncated or rejected by the host under one documented rule, and the plugin
  does not attempt to raise the bound.

### Result presentation

- Results are presented as a bounded, order-preserving set for one owning view,
  oldest retained scrollback first, as returned by the bounded search API. The
  plugin renders only the fields the snapshot and result carry; it does not
  re-derive matches from raw grid access, because no raw grid handle exists.
- The plugin may render a match count, a current-versus-total indicator, and a
  bounded result list in its own overlay, status line, or dedicated view. These
  are presentation-only and must not change content geometry.
- A refresh replaces the presented set; the plugin never appends stale matches
  across refreshes and never renders a result from a prior generation as live.

### Result identity and navigation

- Each result is treated as an opaque, generation-stamped identity: an owning
  view, a stable line id when in scrollback or a live buffer row otherwise, an
  inclusive column span, and a generation. The plugin never uses a raw viewport
  row as identity because resize and reflow shift rows.
- Navigation requests a move to a result identity within the owning view; Core
  clamps the scroll window and performs it. The plugin does not compute absolute
  row targets, does not scroll by raw offset arithmetic, and does not write PTY
  bytes.
- A navigation request whose target is stale or whose owning view no longer
  resolves to a live grid fails closed: no jump, no content read, and the plugin
  discards or refreshes the result set.
- Repeated navigation is deterministic over the finite result set; the plugin
  does not fabricate results or wrap beyond the bound.

### Owner-frame-only highlighting

- Match highlighting is a presentation-only projection requested through the
  host over the owning view's grid. The plugin may select the style and
  distinguish the current navigated match from the remaining matches, but the
  host clips coordinates to the view's column window and translates them with
  the same row mapping as hit testing.
- Highlighting paints only inside the owner's content frame. The plugin cannot
  paint outside the owning view, into another view or panel, at global
  coordinates, or over host chrome; a project-wide or workspace-wide highlight
  is not expressible.
- Highlighting changes no content geometry, writes no Terminal Truth, and
  lingers for no stale generation; when the owning view no longer resolves to a
  live grid, the plugin stops requesting the highlight and the host paints none.

### Search keybindings

- Keybindings are suggestions through the accepted
  `bitty.keymaps.suggest` contract and resolve under the accepted precedence
  (explicit user > workspace > first-party/default > plugin suggestion); chord
  conflicts produce diagnostics for user resolution rather than shadowing.
- The keybinding namespace, the default chord set, and the search command
  identifiers are package policy owned by `CTX-0003` and `W-138`; this page
  fixes only that they are suggestions, capability-neutral, and unable to
  override the user keymap.
- Core's internal search stays usable with zero plugins and in `bitty --safe`;
  the plugin's keybindings are additive and absent in no-plugin mode.

## Copy-mode plugin policy

The copy-mode plugin owns the modal **interaction** policy. Core owns the
selection semantics and the clipboard permission. The plugin never mutates
Terminal Truth and never bypasses the clipboard gate.

### Modal keybindings and cursor movement

- Copy mode is a transient modal interaction over one owning view. It acquires
  keyboard focus through the `W-01` focusable-overlay and transient
  input-capture host API and releases it on cancel, submit or copy, focus
  switch, plugin unload, plugin crash, and Core-side timeout.
- While copy mode holds capture, movement and selection keys produce no PTY
  bytes and do not reach the terminal; capture is revoked by Core and can never
  pin input.
- The plugin owns the modal keymap as policy: typical movement, word and line
  motion, page and half-page motion, start and end, and a yank or copy action.
  Movement operates on the presentation cursor and the view's scroll window,
  not on Terminal Truth; the terminal cursor and modes are unchanged.
- The exact modal keymap, chord defaults, and command identifiers are package
  policy owned by `CTX-0003`; this page fixes only that they are keymap
  suggestions, that modal keys write no PTY bytes, and that the interaction
  composes with, and does not change, mouse modes, focus reporting, alternate
  scroll, or bracketed paste.

### Selection interaction

- The plugin drives selection only through the Core-owned selection API:
  selection get and set carry identity and generation, and the plugin proposes
  an anchor, a focus, and a selection kind. Core validates, normalizes, expands,
  snaps wide characters, reclamps on resize, and truncates on pruning; the
  plugin does not own those rules.
- The accepted kinds are `Simple` (stream or range), `Word`, `Line`, and
  rectangular `Block`; the plugin may choose a kind by gesture policy, but the
  expansion and text-extraction rules stay Core-owned.
- At most one live selection exists, owned by exactly one view. A selection
  without an owner is not representable, and starting a selection in another
  view replaces the live one.
- A stale selection whose line identity, view, or generation no longer resolves
  fails closed: the plugin drops it or re-requests a valid one, and never
  retargets unrelated content.
- Copy mode may coordinate with the search plugin through Core-owned selection
  and navigation, for example bringing a search result into view and selecting
  it for a subsequent copy; the search plugin does not own the selection and
  the copy-mode plugin does not own the search mechanism.
- Selection is presentation state: the plugin never writes the grid, cursor,
  modes, scrollback, or a semantic zone, and no selection becomes persisted
  terminal content.

### Clipboard interaction

- A copy or yank action writes through the Core clipboard permission gate with
  the `clipboard.write` capability. The plugin cannot bypass the gate, the
  authoritative suspicious-paste inspector, or the bound.
- Copied content is exactly the selected text, bounded by
  `CLIPBOARD_MAX_BYTES=8192` with char-boundary truncation and a `truncated`
  flag. A direct user copy is a trusted user gesture; a plugin-initiated copy is
  capability-gated, attributed, and subject to the per-plugin budgets.
- OSC 52 read remains deny-by-default and OSC 52 write remains gated; the
  plugin introduces no ambient clipboard read, no unbounded copy, and no
  clipboard access on the input hot path.
- A plugin without the clipboard capability is denied with a typed failure and
  no partial write; the plugin surfaces the denial rather than retrying
  silently.

## Input capture dependency

Search and copy mode require the `W-01` focusable-overlay and transient
input-capture host API. This document **depends on** that contract and decides
none of it.

- The `W-01` host API is the `bitty-docs` contract for a capability-gated Core
  host mechanism a plugin claims to take exclusive keyboard and overlay focus
  for the duration of one UI interaction ([Issue
  #396](https://github.com/bitty-terminal/bitty-docs/issues/396), under
  `OQ-056`). Its primitive spellings, capture payloads, focus-order rules, and
  timeout values are co-owned and marked provisional here until `W-01` accepts
  them.
- The constraints this document composes with, without redefining them: capture
  is capability-gated, transient, bounded, and revocable on cancel, submit,
  focus switch, plugin unload, plugin crash, and Core-side timeout; it never
  places a plugin callback on the input hot path; it is Core-owned and can
  never pin input; and it opens no second input channel beyond the accepted
  input-pointer and IME direction.
- The Composer capture lifecycle and the Beacon transient-capture rules are the
  consumer shape this page follows: a focusable overlay plus transient capture,
  typed outcomes, and guaranteed release. The plugin must not build on the v1
  non-focusable overlay, which cannot hold focus or receive modal input.
- Until `W-01` lands, the reviewed Core-internal search and copy mode remain the
  only behavior; this plugin-facing surface does not exist yet, and this page
  makes no implementation claim for it.

## Public API requirements

The plugin reaches Core only through public, versioned, capability-gated host
operations. The requirements below are **high level**; the exact function
names, argument tables, result schemas, and error identifiers are delegated to
`W-139` and must be derived from the `W-135` mechanism contract, not invented
here. No requirement widens the accepted v1 surface on its own.

### Snapshot and viewport reads

- The plugin reads Terminal Truth only through a bounded, read-only, per-view
  snapshot: the ordered buffer of retained scrollback plus the live grid, with
  per-row text, column geometry, the owning `ViewId`, and the attached terminal
  identity needed to resolve search and selection.
- A snapshot is scoped to exactly one owning view and its attached terminal. A
  window-wide, workspace-wide, or cross-panel read is refused fail-closed with
  a typed outcome; a failed read returns no data.
- Bounds are finite and enforced before allocation: the host truncates an
  over-limit pattern at a char boundary and clamps or rejects results and
  snapshot requests under one documented rule with the previous state intact.
- The plugin holds no mutable grid handle, no pointer into the grid, and no
  live core object; a snapshot is a copy, never a second state.

### Navigation

- The plugin requests a viewport move to a target identity within one owning
  view. Core clamps the move, changes only the view offset, writes no PTY
  bytes, and never changes content geometry.
- Navigation may synchronize a Core-owned selection to the target so a
  subsequent user copy acts on it, but the selection remains owned by the bound
  view and follows the accepted selection lifecycle.
- A stale target or a missing live grid fails closed with no jump and no
  content read. Navigation is always available and cannot pin input.

### Selection get and set

- Selection get and set carry an identity and a generation. The plugin proposes
  anchor, focus, and kind; Core validates and returns the normalized result or
  a typed `Stale` outcome on a generation mismatch.
- Selection is single-owner: a selection without an owning view is not
  representable, and a set request for a stale or foreign view is refused.
- No selection operation exposes Terminal Truth or a write path into it.

### Clipboard writes

- Clipboard writes go through the permission gate with `clipboard.write`,
  bound by `CLIPBOARD_MAX_BYTES=8192`, char-boundary truncation, and a
  `truncated` flag. The plugin cannot bypass the gate, the bound, or the paste
  inspector.
- Clipboard read, OSC 52 read, and any ambient clipboard path remain outside
  the plugin unless a separate accepted capability grants them; this page
  requires no clipboard read for search or copy mode.

### Cross-cutting API rules

- Every host operation returns a typed outcome and never a bare boolean; a
  denial names the missing capability or the violated rule, and a stale
  reference is rejected by Core, not trusted from the plugin.
- Every snapshot, result, selection, and capture reference carries a
  generation; a re-registered or recycled identity stales prior references and
  a mismatched generation fails closed.
- No operation exposes a direct Terminal Truth handle, a raw grid or PTY
  handle, a GPU or window handle, or a mutable core object.
- No operation invokes a plugin callback on the input, parser, or render hot
  path. Querying, snapshotting, searching, and highlighting run on a safe,
  cold-path boundary.
- Search/copy-mode-facing snapshot, search, selection, clipboard, and capture
  operations are **provisional** until `W-139` binds them and `W-01` accepts
  the capture contract. The accepted v1 `bitty.terminal.snapshot` (visible
  viewport only, no scrollback, `SNAPSHOT_MAX_BYTES`, `terminal.semantic-read`)
  is not sufficient by itself for full scrollback search and is not silently
  widened by this page.

## Capabilities

Each plugin declares the narrow capabilities it needs through the manifest
under the closed v1 family grammar
([Plugin Platform RFC](plugin-platform-rfc.md),
[manifest authority](manifest-capability-authority.md)). Grants are
deny-by-default, bound to the manifest hash, and block capability-increasing
updates pending review.

| Plugin      | Capability intent                                                                                                 | Notes                                                                                       |
| ----------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `search`    | Read-only terminal content access for the bounded search and snapshot surface; `ui.overlay` for the input surface | The read capability spelling is delegated to `W-139`; `terminal.raw-read` is not requested. |
| `copy-mode` | `ui.overlay` for the modal overlay; `clipboard.write` for copy/yank                                               | `clipboard.read` is not requested; OSC 52 read stays deny-by-default and write stays gated. |

- **Deny by default.** Absent from the grant set is denial; there is no
  allow-all identifier and no family-wide wildcard. An undeclared capability
  call fails closed with a typed outcome.
- **No widening.** A plugin cannot invent a capability family or identifier,
  cannot escalate through another plugin's service call, and cannot be granted
  a capability implicitly by workspace configuration. If search or copy mode
  needs a capability not present in the closed set, a successor RFC must add it
  with its own security review; this page does not add one.
- **No private first-party bypass.** The first-party search and copy-mode
  plugins reach Core only through the same public, capability-gated contract a
  third-party plugin uses. There is no private channel, no privileged flag, and
  CI may not add one.
- **High-risk separation.** The plugin does not request `terminal.raw-read`,
  `terminal.manage`, `terminal.input.*`, `ui.protocol-register`, or
  `runtime.*`; a high-risk capability is never bundled with presentation.
- **Safety.** Core search and selection remain usable with zero plugins and in
  `bitty --safe`; the plugin is absent with no partial activation when
  disabled, failed, incompatible, or ungranted.

## Page sets

The search and copy-mode plugins follow the standard per-plugin page set
defined by the repository documentation workflow and
[per-plugin partition](../docs/plugins/README.md): `README.md`, `design.md`,
`schemas.md`, and `evidence.md`. Pages are created only when real content
exists; empty placeholders are not added.

| Page          | `document_type`        | Real-content guidance for search and copy mode                                                                                                          |
| ------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `README.md`   | `index`                | Identity, owning repository, stage, status source, and links to the other pages; never a candidate described as implemented.                            |
| `design.md`   | `specification`        | Scope, UX, keybindings, result presentation, selection interaction, the capability and mechanism/policy split, failure behavior, and alternatives.      |
| `schemas.md`  | `contract`/`reference` | Manifest capability declarations, configuration keys under the plugin namespace, command and keybinding identifiers, and the consumed host/API schemas. |
| `evidence.md` | `register`             | Decision and open-question links, reviews, experiments, and test or release evidence; mark implemented versus verified versus unverified.               |

The packages also reference, without duplicating, the canonical SDK surface
([Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)) and
the terminal-side mechanism contract; plugin pages own only plugin-specific
additions. The package identity, delivery shape, and page content are owned by
`CTX-0003`; this page decides none of them.

## Security review

This document crosses the plugin-to-Core read boundary, the selection and
clipboard trust decisions, the bounded search surface, the capability model,
and the input-capture dependency. Independent security review is required
before this document is promoted beyond draft. The reviewer must confirm:

- the plugins use only the public, capability-gated API, with no private
  first-party bypass, no raw grid or PTY handle, and no input hot-path callback
  (`P0-AC-012`, `P0-AC-015`);
- a snapshot read is bounded, read-only, and per-view; a cross-panel or
  cross-view read is refused, and no result, highlight, or clipboard path
  becomes a cross-panel read or an Event-Bus exposure of another view's content
  (`P0-AC-016`);
- selection and clipboard stay Core-owned and permission-gated: at most one
  live selection per view, copy bounded by `8192` bytes with char-boundary
  truncation, paste inspected by the authoritative inspector, and OSC 52 read
  deny-by-default (`P0-AC-008`, `P0-AC-039`);
- search, snapshot, and highlight work is bounded on pattern, result, row, and
  serialized size, and a stale identity fails closed rather than resolving to
  unrelated content (`P0-AC-014`);
- input capture is Core-owned, transient, bounded, revocable, and guaranteed on
  cancel, submit, focus switch, plugin unload, plugin crash, and Core-side
  timeout, and never places a plugin callback on the input hot path;
- capability declarations are deny-by-default, closed-set, non-widening, and
  official plugins pass the identical model; safe mode and zero-plugin startup
  keep Core search and selection usable (`P0-AC-019`);
- copied search or selection content is treated as terminal observation data
  and never as plugin instructions or authority (`P0-AC-026`).

This document authorizes no implementation; the `W-135` mechanism contract, the
`W-139` SDK surface, and the `CTX-0003` package each require security review
again before their own merge.

## Verification plan

This is a policy and API-requirements specification; it has no executable
verification of its own. Any later implementation that cites it must prove, at
minimum:

1. **Scope escape.** The search plugin bound to view A cannot read, search, or
   navigate view B's grid; an out-of-scope snapshot, search, or navigation
   request is denied with a typed outcome and returns no data.
2. **Stale identity.** A result, selection, or capture invalidated by erase,
   resize, reflow, scrollback prune, or grid replacement fails closed; a
   re-registered identity stales prior references; navigation or copy against a
   stale identity performs no jump and reads no content.
3. **Clipboard denial.** A copy-mode plugin without `clipboard.write` is denied
   with a typed failure and no partial write; an over-limit copy truncates at
   `8192` bytes with a `truncated` flag; OSC 52 read stays deny-by-default and
   write stays gated; `clipboard.read` is absent for both plugins.
4. **Cross-panel read denial.** Output, search, or a highlight on another
   view's grid never refreshes a bound result set or paints; a failed owner
   resolution yields no selection and no highlight.
5. **Input-capture release.** After cancel, submit or copy, focus switch,
   plugin unload, plugin crash, and Core-side timeout, capture is released,
   terminal input is restored, and no captured key reaches the terminal
   unintentionally; modal keys produce no PTY bytes.
6. **Capability model.** An undeclared or unfamilied capability is rejected at
   manifest validation; an official plugin passes the identical model with no
   privileged path; a capability-increasing update blocks pending review.
7. **Safe mode.** With zero third-party plugins and in `bitty --safe`, Core
   search and selection remain usable; the plugins are absent with no partial
   activation.
8. **Bounds.** An over-limit pattern truncates at a char boundary; an over-limit
   result or snapshot request is clamped or rejected before allocation with the
   previous state intact; empty and truncated input never panics.
9. **No hot-path work.** Search, snapshot collection, navigation, highlighting,
   and capture delivery run off the input, parser, and render hot paths, and no
   plugin callback is invoked per keystroke.
10. **No Terminal Truth mutation.** Selection, movement, highlight, and copy
    leave the grid, cursor, modes, scrollback, and semantic zones unchanged,
    asserted by before-and-after snapshot comparison.
11. **Documentation gates.** The repository-local `just check` passes with zero
    issues.

## Alternatives considered

| Alternative                                                       | Trade-off                                                                                                       | Disposition                                                                                    |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Keep search and copy-mode UX inside Core and ship no plugin       | Fewest moving parts, but leaves optional policy in the small core and contradicts ADR 0015 and ADR 0016.        | Rejected; the mechanism stays in Core and the UX policy moves to the extension.                |
| Let the plugin own the search mechanism or selection semantics    | Centralizes matching and copy logic in the plugin, but moves a Core trust and correctness decision out of Core. | Rejected; search, selection semantics, and clipboard permission stay Core-owned.               |
| Give the plugin a live grid handle or a Terminal Truth write path | Simplest read and highlight path, but withdraws Core ownership and opens a cross-panel mutation path.           | Rejected; the plugin reads a bounded snapshot and writes only presentation.                    |
| Build on the v1 non-focusable overlay                             | Reuses an existing slot, but the slot cannot hold focus or receive modal input.                                 | Rejected; the dependency is the `W-01` focusable-overlay and transient input-capture host API. |
| Push highlighting through a global annotation or scene layer      | One uniform paint path, but becomes a cross-panel read and host-chrome mutation.                                | Rejected; highlighting is clipped to the owner's content frame.                                |
| Add a dedicated search or copy-mode capability now                | Avoids a later RFC, but invents a family outside the closed set and pre-empts security review.                  | Rejected; a new family requires a successor RFC, and exact spellings belong to `W-139`.        |
| Let the plugin register a per-keystroke search callback           | Simplest incremental UX, but places plugin code on the input hot path.                                          | Rejected; queries run on a safe boundary and capture produces no hot-path callback.            |

## Affected contracts

| Contract                                                                                                                                                                                                                                                                                 | Effect                                                                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| [Search, Selection, and Snapshots Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/search-selection-contract.md) (`W-135`, `CTX-0091`) (draft)                                                                                                   | Consumed; its snapshot, identity, navigation, selection, clipboard, and bounded-search mechanism is not redefined here.  |
| [Plugin Platform RFC](plugin-platform-rfc.md) (accepted)                                                                                                                                                                                                                                 | Consumed; the closed capability grammar, deny-by-default grants, official parity, and no-hot-path rule are not reopened. |
| [Plugin Manifest and Capability Grammar Authority](manifest-capability-authority.md) (accepted)                                                                                                                                                                                          | Consumed; no capability family or spelling is added by this page.                                                        |
| [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted)                                                                                                                                                                                                      | Consumed; the v1 surface is not silently widened; search/copy-mode host operations are provisional pending `W-139`.      |
| [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (accepted)                                                                                                                                                                                                            | Unchanged; per-plugin budgets bound the search, snapshot, and highlight work.                                            |
| Terminal-side [Input and Pointer Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/input-pointer-rfc.md) and [Composer](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md) (draft/accepted) | Consumed; the selection model, `8192`-byte bound, and `W-01` consumer shape are not redefined.                           |
| Terminal-side [Search, Selection, and Snapshots Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/search-selection-contract.md) (draft)                                                                                                           | Owns the terminal-side mechanism; the plugin policy here must not weaken it.                                             |
| SDK surface `W-139` (`CTX-0066`)                                                                                                                                                                                                                                                         | Gains the high-level snapshot, search, navigation, selection, clipboard, and capture API requirements to bind.           |
| Search/copy-mode package `CTX-0003`                                                                                                                                                                                                                                                      | Gains the plugin policy, capability, and page-set requirements it must implement and document; content stays its own.    |
| Core integration `W-143` (`CTX-0936`) and `W-144` (`CTX-0937`)                                                                                                                                                                                                                           | Gain the plugin-facing requirements the host wiring must satisfy.                                                        |
| [Product plugin roadmap](../product/plugin-roadmap.md) (draft)                                                                                                                                                                                                                           | Records the search and copy-mode plugin rows and their capability sketches; this page does not decide roadmap placement. |

## Open points

None of these is a new global open question; each is parked with its named
owner.

- **Exact host and SDK spellings** for snapshot, bounded search, navigation,
  selection get and set, clipboard write, and capture are owned by `W-139`
  (`CTX-0066`) and derived from `W-135`; the search/copy-mode operations in this
  page are provisional until then.
- **The read capability spelling** for search and snapshot over scrollback is
  delegated to `W-139`; if a dedicated capability is required, a successor RFC
  must add it to the closed set with its own security review.
- **`W-01` host primitive spellings, capture payloads, focus-order rules, and
  timeout values** are owned by `W-01` ([Issue
  #396](https://github.com/bitty-terminal/bitty-docs/issues/396), under
  `OQ-056`); this page references the dependency and decides none of it.
- **Delivery shape, package layout, keybinding namespace, case and regex
  scope, and result presentation** are parked to `CTX-0003` and `W-138`.
- **Selection persistence policy** — whether one persistent selection per view
  is enabled and how search navigation interacts with it — is parked to
  `CTX-0003` and `W-138`.
- **Regex versus literal search and Unicode case-folding scope** (`OQ-074`) and
  **keyboard-selection and copy-mode semantics, and mouse-mode precedence**
  (`OQ-075`) remain Open; this page does not decide them.
- **Numeric bounds for snapshot rows, serialized size, scrollback depth, and
  the capture queue** are co-owned with `W-01`, `W-135`, and `CTX-0003`; they
  must be finite and enforced, but only the pattern, result, and clipboard values are confirmed by current implementation evidence.
- **IME and pointer capture interaction** is owned by the input and IME
  contracts; this page requires no second input channel.
- **Cross-panel or cross-window search and highlight visibility** defaults to
  no; any future visibility is a scoped security decision, not a change here.

## Acceptance criteria

1. The search plugin policy is defined over the Core-owned bounded search API:
   input box UX, result presentation, result identity, navigation, keybindings,
   and owner-frame-only highlighting; the plugin owns policy only and never the
   search mechanism.
2. The copy-mode plugin policy is defined over Core-owned selection semantics:
   modal keybindings, cursor movement, and selection interaction; Core keeps
   selection semantics and clipboard permission; the plugin never mutates
   Terminal Truth.
3. The input-capture dependency references `W-01` ([Issue
   #396](https://github.com/bitty-terminal/bitty-docs/issues/396), `OQ-056`) and
   the Composer and Beacon capture rules without deciding them, and marks the
   dependent host operations provisional.
4. High-level public-API requirements cover bounded per-view snapshot and
   viewport reads, navigation, selection get and set with identity and
   generation, clipboard writes through the permission gate with the `8192`
   byte bound and OSC 52 rules, no direct Terminal Truth handle, and no
   hot-path callback; exact spellings are delegated to `W-139`.
5. The capability declarations each plugin requests are stated with
   deny-by-default, no widening, and no private first-party bypass, and no new
   capability family is invented.
6. The standard plugin page set (`README.md`, `design.md`, `schemas.md`,
   `evidence.md`) is provided as real-content guidance for the search and
   copy-mode packages.
7. The Core search-selection contract, the `W-139` SDK surface, and the
   search/copy-mode package `CTX-0003` are cross-linked without deciding their
   content.
8. Security review and a verification plan with negative-path evidence cover
   scope escape, stale identity, clipboard denial, cross-panel read denial,
   input-capture release, the capability model, safe mode, bounds, no hot-path
   work, and no Terminal Truth mutation.
9. The page is self-contained with no research-repository reference, no record
   number, and no coverage ledger; it describes nothing as implemented, and
   `just check` passes with zero issues.

## P0 Review Sign-off

Not signed. This document is a **draft** plugin-side policy and API
specification. Independent category-owner, docs-curator, and security review
are required before it is promoted beyond draft; the security review above
records the required controls, and no P0 control is changed by this page.

| Role                 | Scope                                                                                        | Requirement                                                                   |
| -------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `architecture-owner` | Search/copy-mode policy, mechanism and policy split, and capability boundaries               | Approve; confirms the plugin owns policy only and Core retains the mechanism. |
| `security-architect` | Snapshot reads, selection, clipboard, bounded search, input capture, capabilities, safe mode | Independent security sign-off required before promotion.                      |
| `docs-curator`       | Metadata, links, terminology, self-containment, and status honesty                           | Approve; confirms schema, discoverability, and draft marking.                 |

## References

- [bitty-plugins-docs#130](https://github.com/bitty-terminal/bitty-plugins-docs/issues/130)
  (CarryCtx `CTX-0073`, plan key `W-138`).
- [Search, Selection, and Snapshots
  Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/search-selection-contract.md)
  (`W-135`, `CTX-0091`) — the authoritative terminal-side mechanism contract.
- [ADR 0015 - Small-Core Extraction
  Boundaries](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0015-small-core-extraction-boundaries.md)
  and
  [ADR 0016 - Execution, Graphics, Accessibility, Storage, and Platform-Service
  Boundaries](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md).
- [Open-question register, OQ-056](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md)
  (the focusable-overlay and transient input-capture host API is v2 scope),
  together with `OQ-074` and `OQ-075`, which stay Open.
- [Plugin Platform RFC](plugin-platform-rfc.md) and [Plugin Manifest and
  Capability Grammar Authority](manifest-capability-authority.md).
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) and
  the [SDK tree index](../sdk/README.md).
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md).
- Terminal-side [Input and Pointer
  Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/input-pointer-rfc.md),
  [Composer Architecture and Host
  API](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md),
  and [Beacon Core Mechanism
  Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md).
- Terminal-side [Performance Budget
  RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/performance-budget-rfc.md)
  and [Compatibility Milestone
  RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/compatibility-milestone-rfc.md).
- [Per-plugin documentation partition](../docs/plugins/README.md) and
  [Plugin documentation template](../docs/plugins/TEMPLATE.md).
- [Security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md),
  [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md),
  [risk register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md),
  and
  [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md).
- [Documentation workflow](../docs/development/documentation-workflow.md) and
  [documentation map](../docs/README.md).
