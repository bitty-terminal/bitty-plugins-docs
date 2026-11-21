---
title: bitty.ui.panel Reference
description: In-place panel content presentation verbs with a Core-resolved target a bounded owner-polled key queue and observation events
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 132
---

# bitty.ui.panel Reference

> Status: **draft**. This page records current host fact only: the surface
> lives in the `bitty` repository at `bitty@944a4b1f` (main head, 2026-11-22)
> and its design contract source is the **draft candidate**
> [Panel GUI Content Presentation](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-gui-content-presentation-candidate.md)
> in bitty-terminal-docs — a candidate is not an accepted contract, and this
> page does not amend the accepted
> [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md), which
> does not name `ui.panel`. The landed slices: the verbs (bitty PR
> [#3267](https://github.com/bitty-terminal/bitty/pull/3267), CTX-1909), the
> input queue, `poll`, and the events (bitty PR
> [#3268](https://github.com/bitty-terminal/bitty/pull/3268), CTX-1911), the
> close-path restore of the displaced PTY (bitty PR
> [#3269](https://github.com/bitty-terminal/bitty/pull/3269), CTX-1912), the
> app producer wire and tick pump (bitty PR
> [#3299](https://github.com/bitty-terminal/bitty/pull/3299), CTX-1929), and
> the user-facing mode drivers (bitty PR
> [#3302](https://github.com/bitty-terminal/bitty/pull/3302), CTX-1931), all
> under [bitty issue #2388](https://github.com/bitty-terminal/bitty/issues/2388).
> This page states no behavior beyond what those sources pin.

## Purpose and scope

`bitty.ui.panel` presents plugin content **in place of** the focused,
terminal-backed panel's content and owns the input that displaced terminal no
longer receives. The plugin supplies a declarative scene and an optional
placement hint; Core resolves the target itself from focus, keeps the
displaced terminal alive and hidden, and restores it on release
(`crates/bitty-runtime/src/runtime/ui_panel.rs` module docs,
`crates/bitty-runtime/src/plugin_runtime/services.rs` `PanelContentBackend`
trait docs, `bitty@944a4b1f`).

The surface is distinct from its neighbours, and mixing them up is the common
misreading:

| Surface                              | Target                                        | Displacement                               |
| ------------------------------------ | --------------------------------------------- | ------------------------------------------ |
| `bitty.ui.mount` / `bitty.ui.update` | Host-owned slots (edge bands, overlay)        | Nothing; slot content only                 |
| `bitty.ui.present.show` / `hide`     | Draws above the focused view                  | Nothing; non-focusable draw                |
| `bitty.ui.overlay.*` (capture)       | The retained `overlay` block                  | Input capture while held                   |
| `bitty.ui.panel.*` (this page)       | The focused, terminal-backed panel **itself** | Terminal content is displaced and restored |
| `bitty.panel.*` (allocation)         | Panels the plugin owns                        | Panel lifecycle, not content               |

The plugin supplies no `PanelId`, `ViewId`, coordinate, size, or surface
handle: the concern split is fixed (`crates/bitty-runtime/src/runtime/ui_panel.rs`
module table, `bitty@944a4b1f`) — Core owns which panel is presented into, the
content swap, the displaced record, the geometry freeze, and the input gate;
the plugin owns the scene it hands over, the handle Core returns, and when it
polls and releases.

## Signature

All four verbs return their refusal as a **second value** instead of raising —
`handle, err` / `ok, err` / `poll, err` — where `err` is the same
`{ class, code, message }` table a raised bridge error carries
(`crates/bitty-lua/src/host.rs` `ui_panel` table comment, `bitty@944a4b1f`;
match on `code`, never on message text).

```lua
bitty.ui.panel.present(component, spec?) -> handle | nil, err?
bitty.ui.panel.update(handle, component) -> boolean | nil, err?
bitty.ui.panel.release(handle, reason?)  -> boolean | nil, err?
bitty.ui.panel.poll(handle, max?)        -> table   | nil, err?
```

The `bitty.ui.panel` table is always present in `bitty.ui`
(`host.rs` `ui.set(ctx, "panel", ...)` at `host.rs:7579`, `bitty@944a4b1f`);
every refusal is per-call, so a host with no presentation path never gains
one from the namespace existing.

`present` validates the component through the same bounded rich-scene reader
`bitty.ui.mount` uses (`read_rich_component`, `host.rs`), so the accepted v1
node vocabulary and the per-generation scene budgets recorded in
[bitty.ui Reference](ui.md) and
[bitty.ui Surface Bounds and Capability Boundary](ui-surface-bounds.md) apply
unchanged; a component this bridge rejects is a typed
`E_UI_COMPONENT_INVALID` before any host call. `update` replaces the layer's
scene and paints nothing else; it leaves the displaced terminal, its winsize,
and the record as they were (`crates/bitty-runtime/tests/ui_panel_api.rs`
rows 8–9, `bitty@944a4b1f`).

`present` returns a **host-derived** handle the caller passes back to `update`,
`release`, and `poll`; it is never supplied by the plugin
(`crates/bitty-runtime/src/plugin_runtime/services.rs` `PanelContentBackend`
trait docs, `bitty@944a4b1f`).

## The closed spec table

`present`'s second argument is a flat table with exactly two admitted fields
(`crates/bitty-lua/src/ui_panel.rs:35` `UI_PANEL_SPEC_FIELDS`,
`bitty@944a4b1f`); `nil` is the same as `{}` (no hint, fixed restore):

| Field       | Values                                          | Meaning                                                                                                                          |
| ----------- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `placement` | `tiled`, `floating`, `fullscreen`, `scratchpad` | Hint only. Recorded, never applied in v1: the panel keeps the placement the user gave it. `band` is not panel content and fails. |
| `restore`   | `previous`                                      | Fixed. The displaced terminal is always restored; the field exists to make the default explicit.                                 |

Anything else — an unknown or extra key, a `placement` outside the four
spellings (including `band`), a `restore` other than `previous`, a
non-string value, or a table deeper than the marshalling limits — is a typed
`E_UI_COMPONENT_INVALID` refusal that changes nothing, never a silently
ignored field (`ui_panel.rs` `read_panel_spec`, `read_placement`,
`read_restore`; the spec table is marshalled under bounded limits of depth 2,
8 nodes, 512 bytes, `ui_panel.rs:134` `PANEL_SPEC_LIMITS`). An ignored key
would be a plugin author believing they declared an authority they do not
hold (`ui_panel.rs` module docs).

`release`'s optional reason is a closed vocabulary too: `released` (the
default when omitted) or `cancelled`, at most 64 bytes
(`ui_panel.rs:50` `UI_PANEL_RELEASE_REASONS`, `ui_panel.rs:53`
`UI_PANEL_RELEASE_DEFAULT_REASON`, `ui_panel.rs:60`
`UI_PANEL_REASON_MAX_BYTES`). A refused reason leaves the presented layer
exactly as it was (`ui_panel.rs` `read_panel_release_reason`).

## Capability

The surface has its own head, `ui.panel`: none of `ui.rich`, `ui.overlay`,
`ui.overlay.focus`, or `ui.present` implies it and it implies none of them
(`crates/bitty-runtime/src/plugin_runtime/services.rs` `UiAccess.panel`
field docs, `bitty@944a4b1f`). The head is in the closed manifest vocabulary
(`crates/bitty-package/src/manifest.rs:375` `CLOSED_CAPABILITY_HEADS`,
`bitty@944a4b1f`), so a manifest naming `ui.panel` validates; grant
lifecycle belongs to the capability contract, not this page.

Enforcement is two-layered, like every other hosted surface
(`crates/bitty-runtime/src/plugin_runtime/mod.rs:3631`,
`bitty@944a4b1f`): the backend is shared only with `ui.panel` generations —
a generation without the grant captures nothing — and each call re-checks
the grant. Checked at `bitty@944a4b1f` (`services.rs` `ui_panel_present`,
`ui_panel_update`, `ui_panel_release`):

| Verb                | Grant order                                                                                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `present`, `update` | `ui.rich` then `ui.panel` (each refused by name with `E_CAPABILITY_DENIED`), then safe mode (`E_UI_UNAVAILABLE`), then backend presence (`E_UI_UNAVAILABLE`) |
| `release`           | `ui.panel` only — releasing paints nothing, so cleanup stays reachable after a `ui.rich` revocation                                                          |
| `poll`              | `ui.panel` before any handle is resolved                                                                                                                     |

Presenting needs both grants because the scene handed over is rich content:
a `ui.panel`-only grant cannot draw an arbitrary tree, and a `ui.rich`-only
grant cannot displace a terminal (`services.rs` `ui_panel_present` docs;
pinned by `ui_panel_api.rs` rows 7 and 14). Safe mode refuses `present` and
`update` with `E_UI_UNAVAILABLE` before the backend is consulted
("safe mode presents no panel content", `services.rs:8155`).

## Errors

Every refusal happens before any mutation and uses the record's spelling
(`crates/bitty-runtime/src/runtime/ui_panel.rs` module table,
`bitty@944a4b1f`):

| `code`                   | `class`      | When                                                                                                                                                                                                      |
| ------------------------ | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `E_UI_COMPONENT_INVALID` | `validation` | An unusable scene, a malformed `spec` or release reason, a non-integer handle, or a non-positive/non-integer `max`; the refusal names the offending key or value, never its content beyond a bounded echo |
| `E_CAPABILITY_DENIED`    | `runtime`    | A missing `ui.rich` or `ui.panel` grant, refused by name (`services.rs` `ui_panel_present`)                                                                                                               |
| `E_TIMEOUT`              | `budget`     | The bounded-call deadline expired; the call commits nothing (`host.rs` `bounded_mutation`)                                                                                                                |
| `E_UI_PANEL_BUSY`        | `runtime`    | The focused panel already presents another owner's layer (`host.rs:1001`)                                                                                                                                 |
| `E_UI_PANEL_NESTED`      | `runtime`    | This generation already presents a layer (`host.rs:1010`)                                                                                                                                                 |
| `E_UI_PANEL_UNAVAILABLE` | `runtime`    | No focused panel, no panel there, or the focused panel is not terminal-backed (`host.rs:1019`)                                                                                                            |
| `E_UI_UNAVAILABLE`       | `runtime`    | No panel host installed (a headless/no-surface host), no backend, or safe mode                                                                                                                            |
| `E_UI_NOT_OWNER`         | `runtime`    | A stale, foreign, previous-generation, or already-released handle; the fence compares owner **and** generation                                                                                            |

One layer per panel and one layer per generation: a second owner on the same
panel is `E_UI_PANEL_BUSY`, and the same generation presenting again is
`E_UI_PANEL_NESTED` (`ui_panel_api.rs` rows 4–5). A handle dies with its
layer — the Core record is released by exactly three paths (the owner's token
release, the generation edge, and panel disposal), and a handle whose record
is gone answers `E_UI_NOT_OWNER` instead of restoring a terminal somebody
else now owns (`crates/bitty-runtime/src/runtime/ui_panel.rs` "Handle
lifetime" docs).

## The displaced terminal and its release paths

While a layer is presented, the displaced terminal stays alive and hidden
with its geometry frozen; nothing on the input path runs plugin code, and no
byte reaches the displaced PTY — the runtime's two keyboard entry points
swallow the key and queue it instead of encoding it, and the first key after
release encodes again (`crates/bitty-runtime/src/runtime/ui_panel.rs` "Input
routing" docs; `ui_panel_api.rs` rows 1 and 12).

`release(handle, reason?)` restores the recorded content kind, prunes the
owner's row, and issues **at most one** restore resize (`ui_panel_api.rs`
row 1). Beyond the owner's call, Core ends a layer through two recorded
paths, and the owner observes each end the same way:

- **Panel disposal.** A layer whose panel closed, was disposed, or whose
  token Core released records the closed-vocabulary reason `closed`
  (`crates/bitty-runtime/src/runtime/ui_panel.rs:100`
  `PANEL_CONTENT_RELEASE_CLOSED`); a generation edge that never saw the
  owner's release records `unloaded` (`:105`
  `PANEL_CONTENT_RELEASE_UNLOADED`). The lifecycle causes the edge names
  are the generation-edge reasons (`suspended`, `reloaded`, `disabled`,
  `unloaded`, `crashed`, `services.rs` `PanelContentBackend::release_owner`
  docs).
- **Close gestures release first** (bitty PR #3269, CTX-1912). The `close_view`
  keymap gesture releases the presented layer and **does not close** — the
  first press restores the terminal the layer was covering, and the next
  press is the close it always was; a refused release blocks the close, because
  closing would retire the panel and kill the shell behind content that
  cannot be dismissed (`crates/bitty-terminal/src/chrome_keys.rs:1604`
  `release_presented_panel_content_for_close`). A plugin panel close releases
  the layer by the panel identity the host still binds before the leaf (and
  the shell behind it) is retired (`chrome_keys.rs` `close_plugin_panel_leaf`).
  A workspace close sweeps every layer the slot presents before the sessions
  are killed and the workspace removed, on both the chord-confirm arm and the
  immediate arm (`crates/bitty-runtime/src/runtime/workspaces.rs:2782`
  `close_workspace_now`; `crates/bitty-runtime/src/runtime/panel_content.rs:403`
  `release_presented_panel_content`).
- **The generation edge.** Suspend, reload, disable, unload, and crash
  release every layer the generation presented, idempotently, with the cause
  recorded on each layer's `panel.content.released` observation
  (`services.rs` `PanelContentBackend::release_owner` docs; `ui_panel_api.rs`
  rows 10–11 — the real `PluginRuntime::suspend`/`reload` edges restore the
  recorded kind and leave the pre-edge handle dead).

A handle Core released for the owning generation answers `released` plus the
reason on the next `poll` (`crates/bitty-runtime/src/runtime/ui_panel.rs:124`
`PANEL_CONTENT_RELEASE_MEMORY_MAX`, a 64-handle memory evicted
oldest-first in handle order).

## The input queue and poll

The queue is Core-owned and lives with the presented layer: Core appends on
the input path and the plugin drains it on its own cold path through
`poll` — Core never calls the plugin to deliver a key
(`crates/bitty-runtime/src/runtime/ui_panel.rs` module docs).

`bitty.ui.panel.poll(handle, max?)` drains up to `max` events, oldest first.
A missing `max` drains 64; an explicit `max` is clamped to the 256-event
queue bound; a non-positive or non-integer `max` is a typed
`E_UI_COMPONENT_INVALID` refusal, not a silent default — a caller that asked
for something impossible must not believe it was honoured
(`crates/bitty-lua/src/ui_panel.rs:115` `UI_PANEL_INPUT_POLL_MAX_DEFAULT`,
`host.rs` `ui_panel.poll` callback, `bitty@944a4b1f`).

The queue is bounded twice, and both bounds evict oldest-first, count the
eviction, and raise the poll result's sticky `overflowed` flag:

| Bound                   | Value                                                                                          | Owner                                                                                                                                                                                                                                                                                                                     |
| ----------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Queued events per layer | 256 (`ui_panel.rs:75` `UI_PANEL_INPUT_QUEUE_MAX`)                                              | This surface                                                                                                                                                                                                                                                                                                              |
| Held text per layer     | 256 KiB (`ui_panel.rs:109` `UI_PANEL_INPUT_BUDGET_BYTES`)                                      | The per-plugin event budget the [Isolation and Resource RFC](../../runtime/isolation-resource-rfc.md) fixes at **RC-5** (PerPlugin 1024 events / 256 KiB aggregate, enforced at `EventPipeline::publish` via DropOldest) — a presented layer holds one plugin's keys, so it is charged against that same per-plugin bound |
| Text per event          | 4096 bytes, truncated at a `char` boundary (`ui_panel.rs:127` `UI_PANEL_INPUT_TEXT_MAX_BYTES`) | A per-event truncation cap shared with the focusable-overlay capture, never a budget                                                                                                                                                                                                                                      |

The byte budget may hold the queue below the event count — with 4096-byte
events exactly `256 KiB / 4096` fit — so the event count is an upper bound,
not a promise (`ui_panel.rs` `UI_PANEL_INPUT_BUDGET_BYTES` reconciliation
docs). The queue is dropped the moment the layer ends, so no key survives
into a panel somebody else owns (`crates/bitty-runtime/src/runtime/ui_panel.rs`
"Input routing" docs). The bounds are pinned by `ui_panel_api.rs` rows 12–13.

A successful `poll` returns a table mirroring the accepted focusable-overlay
poll shape (`crates/bitty-lua/src/host.rs:1172` `PanelPoll`,
`host.rs:3501` `panel_poll_value`, `bitty@944a4b1f`):

| Field        | Meaning                                                                                                                                                                                                                            |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status`     | `"active"` while this generation's layer is live; `"released"` once Core released it                                                                                                                                               |
| `seq`        | Monotonic sequence of the last event delivered to this handle                                                                                                                                                                      |
| `events`     | Drained events, oldest first; each is `{ seq, type, text }` plus `data = { key, ctrl, alt, shift, super }` for a modified key — the same bounded key payload the overlay capture uses, one event shape for one bounded key channel |
| `dropped`    | Key events this layer's queue dropped since present — both bounds count here, so a drop is never silent                                                                                                                            |
| `overflowed` | Sticky once a bound evicted an older event                                                                                                                                                                                         |
| `reason`     | Present only with `"released"`: the closed-vocabulary release cause                                                                                                                                                                |

A `released` poll result carries no events; the drain and the release memory
together are how an owner that missed the close gesture still learns the
layer ended (`ui_panel_api.rs` rows 14–15).

## panel.content.* events

The runtime produces exactly two observation-only events for this surface
(`crates/bitty-plugin-host/src/event.rs:265`–`266`, `bitty@944a4b1f`):

| Event                           | Payload                     | When                                                                                                   |
| ------------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------ |
| `panel.content.released`        | `{ owner, handle, reason }` | A layer ended — owner release, close, disposal, or generation edge — with the closed-vocabulary reason |
| `panel.content.input-available` | `{ owner, handle, queued }` | Level-triggered nudge that keys are queued                                                             |

Payloads carry no content and no key bytes, so the delivery call site hands
each one to the bus without further filtering
(`crates/bitty-runtime/src/plugin_runtime/mod.rs` `pump_panel_content_events`
docs). Delivery is cold-path only: the shipped app installs the
`RuntimePanelContentBackend` at its composition root before activation, over
the shared runtime handle the app owns (`crates/bitty-terminal/src/plugin_runtime.rs:2155`,
bitty PR #3299), and pumps the observations on its cold tick right after the
plugin-timer pump — never the input path — so a handler only observes what
already happened (`crates/bitty-terminal/src/terminal_app.rs:2620`,
`:3254`; `crates/bitty-terminal/src/plugin_runtime.rs:7879` producer-wire
rows). No plugin runtime or no backend answers `0`, and the verbs keep
failing closed with `E_UI_UNAVAILABLE` without one.

Delivery requires the kind in the manifest's `[lazy].events`, the same
precondition `bitty.events.subscribe` enforces, and the pending-observation
buffer between drains is bounded at 64 with drop-oldest — a drop is counted
in the Core-owned `ui_panel_events_dropped` counter, never silent
(`crates/bitty-runtime/src/runtime/ui_panel.rs:116`
`PANEL_CONTENT_EVENT_QUEUE_MAX`). Both events are pinned end-to-end by
`ui_panel_api.rs` row 15.

## Mode moves under a presented layer

A panel that presents a content layer may move between **tiled** and
**floating** while the layer is up, and the layer does not notice: the move
re-places the leaf's frame (a floating layer re-places at the configured
float frame with the elevated `Float` paint tier) while the panel's
identity, its attachment, the content kind, the token, the scene slot, and
the displaced terminal's winsize stay exactly as they were. The mode change
is a layout fact only — no `LogicalRect -> PTY` step runs and no `SIGWINCH`
is delivered for the new frame, and focus is untouched
(`crates/bitty-runtime/src/runtime/panel_content.rs:430`
`set_panel_content_presentation` docs and
`crates/bitty-runtime/tests/panel_content_presentation_modes.rs` rows 1–2,
`bitty@944a4b1f`).

The four placement spellings all **parse**, but only two are answerable at
the single gate every mode change routes through
(`crates/bitty-runtime/src/runtime/panel_content.rs:1022`
`presented_panel_mode_supported`):

| Mode         | Gate answer   | Why                                                                                                                                                                                     |
| ------------ | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tiled`      | Answerable    | The panel's own solver slot; a real placement.                                                                                                                                          |
| `floating`   | Answerable    | The configured float frame with the elevated `Float` tier; a real placement.                                                                                                            |
| `fullscreen` | Refused typed | App-level zoom (`zoom_backup` backup/restore in the host application), not a leaf stamp — a stamp-only mode would paint exactly like the mode it replaced, which is a lie about layout. |
| `scratchpad` | Refused typed | The scratchpad store's hide/slot behavior plus the float frame; a bare stamp would produce a float that never hides.                                                                    |

The refusal is typed (`PanelError::ContentPresentationUnsupported` in the
Core record) and happens at the single gate **before any state moves**: leaf
mode, frame, focus, winsize, and record are all unchanged
(`panel_content.rs:1013`–`1022`; `panel_content_presentation_modes.rs`
rows 4–5). The gate itself runs validate → stamp → record: the stamp goes
through the workspace command registry
(`bitty_ui::apply_presentation_command`), never a direct layout write, and
the record is written only after the stamp, so a refused or out-of-band
state is never reported as changed (`panel_content.rs:444`–`463`). A
redundant request — the mode the leaf already has — is a true no-op, and
release returns the panel to the mode the layer displaced, measured before
the restore resize, so it never overwrites a mode a third party stamped
after the layer moved (`panel_content.rs:458`–`463`;
`panel_content_presentation_modes.rs` rows 6–8).

There is no plugin-facing spelling for a mode move: `bitty.panel.set_presentation`
is grant-gated on `panel.focus` and then fails closed with
`E_NOT_IMPLEMENTED`, because no plugin-facing present hands a plugin the
`PanelContentToken` that fences a change
(`crates/bitty-runtime/src/plugin_runtime/services.rs:8290`;
`crates/bitty-runtime/src/runtime/panel_content.rs` "Not in this slice"
docs). The mode changes a presented layer actually receives are user-driven,
and both route through the same single gate (bitty PR #3302, CTX-1931,
merged 2026-11-22): the `bitty ctl panel mode p:N tiled|floating [TOKEN]`
debug verb (`crates/bitty-terminal/src/ctl/request.rs:544`) and the
`ctrl+alt+a` `toggle_panel_mode` keybind, a `global`-only chord with the
alias `panel_mode_toggle` (`crates/bitty-config/src/keymap.rs:1048`). No
plugin surface changed with them; the plugin's role is that its layer,
handle, token, and content survive the user's move
(`panel_content_presentation_modes.rs` rows 1–3).

The `spec.placement` hint is not a fourth driver: it is recorded and never
applied in v1 — the panel keeps the placement the user gave it
(`ui_panel.rs` module docs; `ui_panel_api.rs` row 2).

## Example

Mapped to the covering rows in `crates/bitty-runtime/tests/ui_panel_api.rs`
(`bitty@944a4b1f`; rows 1, 12–13, 15 — the same shapes the producer-wire
rows in `crates/bitty-terminal/src/plugin_runtime.rs:7879` drive over the
real discovery path):

```lua
-- Manifest: capabilities = { ui.rich = true, ui.panel = true }
-- and panel.content.released in [lazy].events to observe the release.

bitty.commands.register({ id = "present", title = "Present", run = function()
  local handle, err = bitty.ui.panel.present(
    { kind = "Text", text = "panel-content-wire" }, {}
  )
  if err then return "ERR:" .. err.code end
  -- The hint in {} is recorded, not applied; the panel stays where the
  -- user put it.
  local poll, perr = bitty.ui.panel.poll(handle) -- drains up to 64
  if perr then return "ERR:" .. perr.code end
  if poll.status == "released" then return "gone:" .. poll.reason end
  return "OK:" .. tostring(#poll.events)
end })

bitty.events.subscribe("panel.content.released", function(payload)
  -- payload = { owner, handle, reason } — no content, no key bytes.
end)
```

The first `present` after the user focuses a terminal-backed panel returns a
handle; a `poll` before any key press returns zero events with
`overflowed = false`; `bitty.ui.panel.release(handle)` (reason defaults to
`"released"`) restores the displaced terminal, and the release surfaces both
on the next poll of the dead handle and as one `panel.content.released`
observation on the app tick.

## Compatibility and versioning

The accepted [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
does not name `ui.panel`, `bitty.ui.panel`, `panel.content.released`, or
`panel.content.input-available`: the surface was added by Core under
bitty#2388 with its contract source the draft candidate named in the status
block, and this page records it as current host fact without amending the
accepted contract, on the same rule the `ui.present` section of
[bitty.ui Reference](ui.md) records a post-ratification head. The candidate
contract is not accepted: nothing here authorizes a compatibility-guaranteed
behavior, and the `1.x` stability policy of the accepted RFC has not been
invoked by this surface. Error codes and constants named here are the
current host spellings pinned to `bitty@944a4b1f`; the release reasons, the
event payloads, and the two answerable modes are closed vocabularies, and a
future acceptance of the candidate contract — not this page — is what would
make them compatibility promises.

## References

- `bitty` `crates/bitty-lua/src/ui_panel.rs` (`bitty@944a4b1f`; the closed
  `spec` table: `UI_PANEL_SPEC_FIELDS:35`, `UI_PANEL_PLACEMENTS:38`,
  `UI_PANEL_RESTORE_PREVIOUS:41`; release reasons
  `UI_PANEL_RELEASE_REASONS:50`, default `:53`, 64-byte bound `:60`; queue
  bounds `UI_PANEL_INPUT_QUEUE_MAX:75`, `UI_PANEL_INPUT_BUDGET_BYTES:109`,
  `UI_PANEL_INPUT_POLL_MAX_DEFAULT:115`,
  `UI_PANEL_INPUT_TEXT_MAX_BYTES:127`; `PANEL_SPEC_LIMITS:134`)
- `bitty` `crates/bitty-lua/src/host.rs` (`bitty@944a4b1f`; the `ui_panel`
  table callbacks `present`/`update`/`release`/`poll` ending at
  `host.rs:7430`/`:7472`/`:7524`/`:7577` and registered at `:7579`;
  `E_UI_PANEL_BUSY:1001`, `E_UI_PANEL_NESTED:1010`,
  `E_UI_PANEL_UNAVAILABLE:1019`; `OverlayInput:1080`; `PanelPoll:1172`;
  `panel_poll_value:3501`)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (`bitty@944a4b1f`; `PanelContentBackend` trait `:909` with
  `release_owner`/`drain_events`; call-time gates `ui_panel_present:8136`,
  `ui_panel_update`, `ui_panel_release`; `panel_set_presentation:8290`)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/mod.rs`
  (`bitty@944a4b1f`; the `ui_panel` grant flag `:3371`; backend shared only
  with granted generations `:3631`; `set_panel_content_backend:1886`;
  `pump_panel_content_events` and its `{ owner, handle, reason }` /
  `{ owner, handle, queued }` payloads `:1901`)
- `bitty` `crates/bitty-runtime/src/runtime/ui_panel.rs` (`bitty@944a4b1f`;
  the module's ownership, input-routing, typed-refusal, and handle-lifetime
  tables; `PANEL_CONTENT_RELEASE_CLOSED:100`,
  `PANEL_CONTENT_RELEASE_UNLOADED:105`,
  `PANEL_CONTENT_EVENT_QUEUE_MAX:116`, `PANEL_CONTENT_RELEASE_MEMORY_MAX:124`;
  `PanelContentEvent`)
- `bitty` `crates/bitty-runtime/src/runtime/panel_content.rs`
  (`bitty@944a4b1f`; `release_presented_panel_content:403`;
  `set_panel_content_presentation:430`; the single mode gate
  `apply_panel_content_mode_command` and
  `presented_panel_mode_supported:1022`)
- `bitty` `crates/bitty-runtime/src/runtime/workspaces.rs`
  (`bitty@944a4b1f`; `workspace_close_at:2756`, `close_workspace_now:2782`)
- `bitty` `crates/bitty-terminal/src/chrome_keys.rs` (`bitty@944a4b1f`;
  `release_presented_panel_content_for_close:1604`,
  `close_plugin_panel_leaf`; the `toggle_panel_mode` gesture `:3070`)
- `bitty` `crates/bitty-terminal/src/terminal_app.rs` and
  `crates/bitty-terminal/src/plugin_runtime.rs` (`bitty@944a4b1f`; the
  producer wire `plugin_runtime.rs:2155` and the cold-tick pump
  `terminal_app.rs:2620` / `:3254`, bitty PR #3299; the composition-root
  rows `plugin_runtime.rs:7879`)
- `bitty` `crates/bitty-terminal/src/ctl/request.rs` (`bitty@944a4b1f`;
  the `panel mode p:N tiled|floating [TOKEN]` wire `:544`)
- `bitty` `crates/bitty-config/src/keymap.rs` (`bitty@944a4b1f`; the
  `ctrl+alt+a` `toggle_panel_mode` default chord `:1048`)
- `bitty` `crates/bitty-package/src/manifest.rs` (`bitty@944a4b1f`;
  `ui.panel` in `CLOSED_CAPABILITY_HEADS:375`)
- `bitty` `crates/bitty-runtime/tests/ui_panel_api.rs` (`bitty@944a4b1f`;
  the fifteen contract rows: present/release round trip, recorded hint,
  typed refusals, two-grant capability gating, queue bounds, remembered
  release, and both events) plus
  `crates/bitty-runtime/tests/panel_content_presentation.rs` (close-path
  restore rows, bitty PR #3269) and
  `crates/bitty-runtime/tests/panel_content_presentation_modes.rs`
  (the eight mode rows)
- `bitty` `crates/bitty-plugin-host/src/event.rs` (`bitty@944a4b1f`;
  `panel.content.released:265` / `:301`,
  `panel.content.input-available:266` / `:302`)
- [Panel GUI Content Presentation](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-gui-content-presentation-candidate.md)
  (bitty-terminal-docs, **draft candidate**, decision-ready — the design
  contract source named by the implementation; not accepted)
- [Isolation and Resource RFC](../../runtime/isolation-resource-rfc.md)
  (accepted; RC-5 PerPlugin 1024 events / 256 KiB aggregate — the budget the
  presented layer's key queue is charged against)
- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (accepted; does not name this surface — see
  [Compatibility and versioning](#compatibility-and-versioning))
- [bitty.ui Reference](ui.md) and
  [bitty.ui Surface Bounds and Capability Boundary](ui-surface-bounds.md)
  (draft; the mount surface whose node vocabulary and scene budgets the
  presented component inherits, and the error-table reading rules)
- [Lua Reference Overview](overview.md) (draft; BridgeError contract and
  capability-gating reading rules)
- Issue tracking: [bitty issue #2388](https://github.com/bitty-terminal/bitty/issues/2388)
  (the epic this surface lands under; this page is its author-guide half —
  residue item 4)
