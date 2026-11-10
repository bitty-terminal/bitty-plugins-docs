---
title: DevTools GUI Live View (Candidate)
description: Candidate decision record for a mountable DevTools live view built only from shipped Plugin API v1 spellings with settled handle contention refresh budget and grant decisions
category: extensibility
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 31
---

# DevTools GUI Live View (Candidate)

> Status: **draft candidate** — not **Accepted**, not **Implemented**, not
> **Verified**, and not normative. This record decides the open design
> questions for an optional GUI face on `bitty-featured.devtools`, so that the
> implementation is written against a settled contract instead of being argued
> in code review. It adds no Core API: every spelling it uses is a shipped
> Plugin API v1 function, error code, capability identifier, event kind, or
> budget, and each is cited to its owning source. A decision recorded here is
> not an implementation claim; nothing in this repository ships the view today.

## Purpose and scope

`bitty-featured.devtools` is a read-only inspector with seven commands and a
notification-shaped output. The open request is an optional GUI face: a
mountable live view that shows the same inspect data as tables plus a live
trace stream, refreshes about once per second, can be paused and resumed, and
keeps the trace backend's dropped-record counter visible. The constraint that
makes the request interesting is that the view must be built **only** from
Plugin API v1 spellings that already ship — the success condition is proving no
new Core API was needed, and treating any real gap as a separate Core issue
rather than as scope here.

Three design questions were left open by the request because they are contract
decisions, not coding details: what happens to the single trace handle the
existing commands share, what the refresh loop is allowed to cost, and what
happens to the existing commands when the UI grants are absent. Two further
questions follow from the request's own acceptance criteria (pause/resume
handle lifetime, and how a per-drain dropped counter becomes a visible session
total). This record settles all five.

In scope:

- W-1 single-handle contention: one decision, with the resulting command
  semantics.
- W-2 refresh budget: default cadence, configuration, floor, pause rule, and a
  per-tick cost model with the ceilings it must stay inside.
- W-3 grants: the consent consequence of adding `ui.*` heads and the
  grant-less degradation path.
- W-4 pause, resume, and the mount lifecycle, including what "close" means when
  the surface has no unmount.
- W-5 dropped-record accounting across drains.
- The `SceneNode` tree the view emits, its bounds, and the failure/denial
  modes with their observable outcomes.

Out of scope and owned elsewhere:

- the closed slot set, the v1 node subset, the `block_id` handle model, and the
  `ui.rich` / `ui.overlay` / `ui.overlay.focus` / `ui.present` gates (accepted,
  [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md);
  current host facts in the [bitty.ui reference](../sdk/reference/ui.md));
- the `bitty.debug.*` read-only backend: trace ring bounds, redaction, the
  per-plugin handle cap, and drain-on-read (Core implementation, cited by
  path below; the protocol-level counterpart is the
  [DevTools RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/devtools-rfc.md),
  which owns the JSONL debug protocol and the observability pipeline, not the
  Lua surface this record uses);
- timer and task caps, one-shot semantics, and event queue budgets (accepted,
  [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) and
  [Plugin Platform RFC](../specifications/plugin-platform-rfc.md));
- panel identity, focus routing, and the exclusion of `register_panel` from
  plugin API v1 (accepted, terminal-side Panel Runtime RFC);
- per-slot multiplicity and conflict resolution (candidate,
  [Plugin UI Slot Inventory](plugin-ui-slot-inventory-candidate.md));
- reusable composition recipes over the v1 nodes (candidate,
  [Plugin UI Component Catalog](ui-component-catalog-candidate.md)).

## Normative sources this specification must not weaken

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (accepted): the seven-name closed slot set, the v1 node subset (`Text`,
  `Row`, `Column`, `List`), the opaque generation-owned `block_id`, the
  `ui.rich` / `ui.overlay` gates, the RC-4 caps (64 tasks / 32 timers per
  plugin), one-shot timers in v1, and the closed event name set.
- [bitty.ui reference](../sdk/reference/ui.md) (draft; current host fact at
  `bitty@7d98f53a` and `bitty@8108e4f7`): the hosted slot table, band-only
  interaction routing, the per-generation bounds table (depth 16, 2048 nodes,
  256 KiB text per block, 64 retained blocks, 2 MiB retained string bytes), the
  `ui.mount` gate order, the presentation-only overlay contract (admission,
  frame budget, `Esc` dismissal, `overlay.dismissed`), and the error-code
  table.
- [bitty.tasks and bitty.timers reference](../sdk/reference/tasks-timers.md)
  (draft; current host fact): `bitty.timers.create` / `cancel`, the
  `REGISTRATION_MAX_TIMERS` (64) capture bound, `E_DEF_LIMIT` on the 65th
  create, the 86 400 000 ms delay ceiling, and one-shot semantics.
- [bitty.events reference](../sdk/reference/events.md) (draft; current host
  fact): the closed v1 kind set, the declared-kind precondition, the
  activation-only subscription rule, and `REGISTRATION_MAX_EVENTS` (256).
- [bitty.settings reference](../sdk/reference/settings.md) (draft; current host
  fact): `bitty.settings.get` reading a manifest `[settings]` declaration, with
  `int` as an accepted kind.
- [bitty.notify reference](../sdk/reference/notify.md) (draft; current host
  fact): the host acceptance boolean and the host-side rate policy that can
  decline a notification without error.
- [Plugin Platform RFC](../specifications/plugin-platform-rfc.md) (accepted):
  deny-by-default grants, the grant-lifecycle update rule (added capabilities
  block automatic update and require a permission-diff approval, R-016), and
  the three-level queue budgets.
- [Isolation Resource RFC](../runtime/isolation-resource-rfc.md) (accepted):
  `RC-4`, 64 tasks and 32 timers per plugin, refusal above the cap with no
  burst queueing.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  (accepted): the accepted scene bounds `SCN-1`..`SCN-5` (2048 nodes, depth 32,
  256 KiB text per block, 2 MiB aggregated rich bytes per terminal, 64 blocks
  per terminal).
- [Security Overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md):
  invariant 3 (presentation never Terminal Truth), invariant 4 (no hot-path
  execution), invariant 7 (bounded inputs).

## Terminology

| Term               | Meaning in this document                                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| Command trace      | The `bitty.debug.trace` handle owned by `devtools.trace` and drained by the `trace-dump` / `trace-stop` commands.                     |
| View trace         | A second `bitty.debug.trace` handle opened and drained by the live view, independent of the command trace.                            |
| Surface            | The `overlay`-slot block the view mounts and presents; one per plugin generation.                                                     |
| Presented / hidden | Whether `bitty.ui.present.show` currently draws the surface. Hidden is not the same as closed: the block is retained either way.      |
| Tick               | One timer callback: one `debug.inspect`, one `debug.trace_get`, one `ui.update`.                                                      |
| Session drop total | The sum of every `dropped` count the view trace has reported, accumulated across drains because the host resets the counter on drain. |
| Degraded run       | A plugin generation without the `ui.*` grants: commands work, the view does not exist.                                                |

## What DevTools does today

The plugin registers seven commands from `init.lua`: four `debug.inspect`
targets (`plugins`, `commands`, `events`, `grants`) and
`trace-start` / `trace-dump` / `trace-stop`
(`plugins/devtools/lua/devtools/init.lua:119-210`, `bitty-plugins@2da1c87`).
Its manifest requests exactly `debug.inspect`, `debug.trace`, and
`platform.notify`, and states that `debug.control` and every `ui.*` identifier
are intentionally absent (`plugins/devtools/bitty-plugin.toml` "capabilities").

Output is a bounded string, not a scene: `format.lua` renders at most 15 item
or record lines per body (`MAX_LINES`), 64 bytes per field
(`MAX_FIELD_BYTES`), 112 bytes per line (`MAX_LINE_BYTES`), 2048 bytes per body
(`MAX_BODY_BYTES`), and derives a one-line notice capped at 256 characters
(`NOTIFY_MAX_CHARS`) for `bitty.notify.show`
(`plugins/devtools/lua/devtools/format.lua:16-40`, `:231-279`). Every host call
is wrapped in `pcall` and a failure becomes a bounded error string, never a
raise (`init.lua:58-80`).

`devtools.trace` is a two-state machine over **at most one** handle
(`plugins/devtools/lua/devtools/trace.lua:1-18`): `start` refuses while tracing
with the plugin-local `E_TRACE_ACTIVE`, `dump` drains the host buffer,
`stop` drains then closes, and a nil drain resets the machine to idle with the
plugin-local `E_TRACE_GONE` (`trace.lua:78-169`).

## Why a GUI face needs no new Core API

Each capability the view needs already has a shipped spelling. This table is the
whole argument; if a row were missing, the request would become a Core issue.

| Need                                      | Shipped spelling                                                              | Owning source                                                                                                                                        |
| ----------------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Emit a declarative tree                   | `bitty.ui.mount(slot, component)`, `bitty.ui.update(block_id, component)`     | [bitty.ui reference](../sdk/reference/ui.md) "Signature"; bridge `ui` table at `crates/bitty-lua/src/host.rs:6276` (`bitty@23ff0f6c`)                |
| Floating, non-modal panel                 | `overlay` slot + `bitty.ui.present.show` / `hide`                             | [bitty.ui reference](../sdk/reference/ui.md) "Presentation-only overlay (`ui.present`)"                                                              |
| Detect `Esc` dismissal                    | `overlay.dismissed` observation via `bitty.events.subscribe`                  | [bitty.ui reference](../sdk/reference/ui.md) "`overlay.dismissed` observation"; [bitty.events reference](../sdk/reference/events.md) closed kind set |
| Periodic refresh                          | `bitty.timers.create(delay_ms, callback)` / `bitty.timers.cancel(timer_id)`   | [bitty.timers reference](../sdk/reference/tasks-timers.md) "Signature"                                                                               |
| Read lifecycle, registrations, own grants | `bitty.debug.inspect(target)` (already granted)                               | `crates/bitty-runtime/src/plugin_runtime/debug.rs` `DebugView` module docs; `MAX_INSPECT_ITEMS` at `:66`                                             |
| Read trace records                        | `bitty.debug.trace(opts)` / `bitty.debug.trace_get(handle)` (already granted) | `crates/bitty-lua/src/host.rs:2623-2640` (`debug_trace_get` contract); `debug.rs:335-393` (`parse_trace_opts`)                                       |
| Configure the cadence                     | `bitty.settings.get(key)` over a manifest `[settings]` `int`                  | [bitty.settings reference](../sdk/reference/settings.md) "Signature and declared keys"                                                               |
| Bounded text                              | existing `format.lua` bounds, reused unchanged                                | `plugins/devtools/lua/devtools/format.lua:16-40`                                                                                                     |

Two consequences follow, and both are decisions in this record rather than
assumptions:

1. `bitty.ui` has **no unmount**. The bridge `ui` table exposes `mount`,
   `update`, `overlay` (`acquire` / `update` / `release` / `show` / `hide` /
   `poll`), `present` (`show` / `hide`), `targets`, and `labels`
   (`crates/bitty-lua/src/host.rs:6276-6410`, `bitty@23ff0f6c`). A mounted block
   lives for its generation, so "close the view" cannot mean "destroy the
   block" (see [W-4](#w-4-pause-resume-and-mount-lifecycle)).
2. Scene-node pointer routing is band-only. `on_click` / `on_scroll` bindings
   dispatch from `top` / `bottom` only; on an overlay they are accepted and
   retained but not routed, because no overlay hit-test contract exists
   ([bitty.ui reference](../sdk/reference/ui.md) "Band-only interaction routing
   and overlay capture"). A floating view therefore cannot have clickable
   pause/resume controls (see [W-2](#w-2-refresh-budget)).

## W-1 single-handle contention

**Decision: the view opens its own second trace handle. The command trace is
never touched.**

Sharing the command handle was the tempting option because the plugin already
owns a session machine for it, but `bitty.debug.trace_get` is drain-on-read: it
returns `{ records, dropped }` and **empties the buffer**, and the `dropped`
counter resets after being reported
(`crates/bitty-lua/src/host.rs:2623-2640`; `TraceHub::drain` takes the counter
with `std::mem::take`, `crates/bitty-runtime/src/plugin_runtime/debug.rs`).
One handle therefore has exactly one reader. Two readers over one handle means
one of them loses records, and the request's own acceptance criterion — the
existing commands behave identically whether the view is open or closed — could
not hold.

### Resulting command semantics

The view calls `bitty.debug.trace` a second time and keeps its handle in its own
slot. The hub fans out **a copy per matching trace** rather than splitting a
stream: the runtime records each delivered event once, before fan-out, and every
live trace whose owner declares the kind and whose filter matches receives its
own copy in a drop-oldest ring
(`crates/bitty-runtime/src/plugin_runtime/debug.rs` module docs and
`TraceHub::record`). The documented sharing rules are:

1. **Two readers, two handles, no stealing.** Each drain empties only the
   handle it addresses. `trace-dump` and `trace-stop` keep draining the command
   handle and report exactly what they report today.
2. **Duplicate recording, not divided recording.** An event matching both
   filters is stored twice, once per handle. The cost is bounded by each ring,
   not by the event rate (see [W-2](#w-2-refresh-budget)).
3. **Independent drop accounting.** Each handle counts its own drops. A view
   paused long enough to overflow its ring inflates the session drop total and
   never disturbs the command trace's counter.
4. **The view never closes the command handle.** `view-close` stops the view
   trace only; `trace-stop` remains the only closer of the command trace.
5. **The view's filter defaults to the command filter when a command trace is
   running**, and to "all declared kinds" otherwise, so the two streams agree
   unless the user deliberately starts a narrowed trace.
6. **The per-plugin cap is respected by construction.** Core allows
   `MAX_TRACES_PER_PLUGIN` = 4 open traces per plugin and refuses the next with
   `E_DEF_LIMIT` (`crates/bitty-runtime/src/plugin_runtime/debug.rs:53`,
   `:562-580`). The plugin uses at most 2: the command trace and the view trace.
   A second `view-open` while open is a no-op returning the current state, so
   the view cannot consume the third and fourth slots.

### Failure mode

`view-open` can fail to get a handle when the cap is already reached (four
traces open by another code path in the same plugin — not reachable from the
shipped commands, but reachable after a future command adds one), when the
`debug.trace` grant is missing, or when the bridge function is absent. Each is a
bounded string in the view header and a notice, and the view still runs: the
inspect tables refresh on the tick and the trace section reads
`trace unavailable: <code>`. No command changes behaviour and nothing raises.

### Why the other option was rejected

**Rejected: the live view owns the command handle while open, and
`trace-dump` / `trace-stop` report an `E_TRACE_ACTIVE`-adjacent state.** It is
simpler in one respect — one handle, one drain — but it makes the view a
destructive observer of a command surface: `trace-dump` would return fewer
records or a refusal depending on whether a window is open, which is precisely
the behaviour the request's acceptance criterion forbids. It also leaves the
pause state incoherent: a paused view that keeps the handle keeps blocking the
commands, and a paused view that releases it loses every record produced during
the pause, so "pause" would silently mean "stop tracing". A second handle costs
one extra copy per event and buys unchanged command semantics; the cap leaves
two unused slots, so the cost is affordable and the semantics are not
negotiable.

## W-2 refresh budget

**Decision: 1000 ms default, configurable through a manifest setting, floored at
250 ms, and the tick only runs while the surface is presented.**

### Cadence

- **Default 1000 ms.** One tick per second is the request's proposal and is the
  rate at which the inspect tables change in practice; the trace stream reads as
  a rolling window rather than a scroll.
- **Configurable.** `bitty.settings.get("refresh_ms")` over a manifest
  `[settings]` entry declared as `int` ([bitty.settings
  reference](../sdk/reference/settings.md)). No new configuration mechanism: the
  manifest declaration is the allowlist and the user file supplies the value.
- **Floor 250 ms, ceiling 60 000 ms.** A value outside the range is clamped to
  the nearest bound and the **effective** value is rendered in the view header,
  so the user sees what is actually running rather than what was requested. The
  host's own ceiling (86 400 000 ms, one day) is far above the plugin's, so the
  plugin bound is the binding one
  ([bitty.timers reference](../sdk/reference/tasks-timers.md) "Limits").

The floor is a work bound, not a taste call. A tick's worst case is one
`debug.inspect` snapshot (≤ 1024 rows), one drain of ≤ 256 records, and one
scene encode of ≤ 8 KiB. At 250 ms that is 4 drains and 4 encodes per second;
at 16 Hz it would be 16, and the drain is the expensive half because it encodes
and clones every record it returns. 250 ms keeps the steady-state encode budget
at or below the size of the trace ring itself (1 MiB) per second.

### The tick is a re-armed one-shot, not a repeating timer

v1 timers are one-shot; repeating timers are a `1.x` minor addition
([Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
"Tasks and timers"). The accepted idiom is a self-re-arming chain: a callback
may call `bitty.timers.create` again, and the pump prunes the fired row **before**
invoking the callback, so the re-arm sees a free slot even at the cap
(`crates/bitty-runtime/src/plugin_runtime/mod.rs:976-987` and `:2251-2295`;
pinned by `at_cap_repeating_chain_reuses_freed_slot` and
`ten_plan_repeating_chains_survive_past_15s` in
`crates/bitty-runtime/tests/plugin_timers.rs:873`, `:979`). The chain holds
exactly **one** live timer at a time, so it uses 1 of the 32 `RC-4` live timers
([Isolation Resource RFC](../runtime/isolation-resource-rfc.md) `RC-4`) and 1 of
the 64 bridge capture slots
([bitty.timers reference](../sdk/reference/tasks-timers.md) "Limits").

A failed re-arm (`E_DEF_LIMIT`, or `E_BRIDGE_ABSENT` on an older host) is
reported once in the header and ends the chain without raising; the view keeps
its last frame and stays open. Timers fire on the cold-tick pump only, which the
application calls before present and never on the input hot path
(`mod.rs:2251-2263`), so the tick inherits the no-hot-path invariant.

### Pause rule

The tick runs only while the surface is **presented**. Two triggers stop it:

1. an explicit `view-pause` command;
2. the surface ceasing to be presented — `view-close`, or an `Esc` dismissal
   observed as `overlay.dismissed` with `reason = "escape"`
   ([bitty.ui reference](../sdk/reference/ui.md) "`overlay.dismissed`
   observation").

Window-level blur is **not** a trigger, because v1 has no signal for it: the
closed `EventKind` set contains no window-focus, visibility, or occlusion kind
([bitty.events reference](../sdk/reference/events.md) "Limits"), and the
presentation overlay follows the focused view rather than reporting window
focus. "Hidden" is therefore defined as "not presented" in this record, and
blur-based pausing is left to an owner decision (see [Open
points](#open-points)) instead of being invented.

### Per-tick cost model

| Step         | Calls per tick | Bound per call                                                                                                             | Owning source                                                                                                                                                                                                              |
| ------------ | -------------- | -------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inspect      | 1              | One target only (the selected one), `MAX_INSPECT_ITEMS` = 1024 rows, `truncated` flag reported                             | `crates/bitty-runtime/src/plugin_runtime/debug.rs:66`, `:222-231`                                                                                                                                                          |
| Trace drain  | 1              | `max_events` = 256 on the view trace, so ≤ 256 records; ring also capped at `TRACE_BUFFER_MAX_BYTES` = 1 MiB               | `debug.rs:41-64` (default 1000, limit 10 000, min 1), `debug.rs:352-362` (`max_events` accepted on the Lua `trace` opts)                                                                                                   |
| Scene update | 1              | ~40 nodes, depth 4, ≤ 8 KiB text — inside depth 16 / 2048 nodes / 256 KiB text / 64 blocks / 2 MiB retained per generation | [bitty.ui reference](../sdk/reference/ui.md) "Per-generation bounds"; `SCN-1`..`SCN-5` in [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md) |
| Notification | 0              | The tick never calls `bitty.notify.show`                                                                                   | `plugins/devtools/lua/devtools/init.lua:58-68`                                                                                                                                                                             |

Three bridge calls per tick, twelve per second at the floor, three per second at
the default. The ceilings this stays inside:

- **`RC-4`**: 32 live timers per plugin, refusal above the cap with no burst
  queueing ([Isolation Resource RFC](../runtime/isolation-resource-rfc.md)
  `RC-4`) — the view uses 1.
- **Bridge capture**: `REGISTRATION_MAX_TIMERS` = 64
  ([bitty.timers reference](../sdk/reference/tasks-timers.md) "Limits") — 1.
- **Trace**: `MAX_TRACES_PER_PLUGIN` = 4, `TRACE_MAX_EVENTS_LIMIT` = 10 000,
  `TRACE_BUFFER_MAX_BYTES` = 1 MiB per trace (`debug.rs:41-64`) — 1 view handle
  at 256 records.
- **Event pipeline**: `PerSubscription` 64 events / `PerPlugin` 1024 events and
  256 KiB / `Global` 8192 events and 2 MiB
  ([Plugin Platform RFC](../specifications/plugin-platform-rfc.md) "Delivery,
  ordering, batching, and coalescing" item 6) — the view takes **one**
  subscription (`overlay.dismissed`), so one 64-deep queue and no aggregate
  pressure.
- **Scene**: `SCN-1`..`SCN-5` and the stricter bridge budgets
  ([Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md);
  [bitty.ui reference](../sdk/reference/ui.md) "Per-generation bounds") — a
  ~40-node tree.

### The notify/update rate policy question

There is no documented per-call rate policy on `ui.update`: the `ui.mount` /
`ui.update` error vocabulary is `E_UI_COMPONENT_INVALID`, `E_TIMEOUT`,
`E_CAPABILITY_DENIED`, `E_UI_BLOCK_BUDGET`, plus `E_UI_CLAIM_REQUIRED` and
`E_UI_UNAVAILABLE` on mount
([bitty.ui reference](../sdk/reference/ui.md) "Errors"), and no rate code
appears in it. The only rate policy on this path is the **notify** one, which is
host-side and declines a notification by returning `false` rather than raising
([bitty.notify reference](../sdk/reference/notify.md) "Returns"). The tick does
not call `notify` at all, so the policy is off the tick path by construction;
`view-open`, `view-close`, `view-pause`, `view-resume`, and `view-target` each
send at most one notification, and a declined notification is already swallowed
by `show` (`init.lua:58-68`).

## W-3 grants

**Decision: add `ui.rich`, `ui.overlay`, and `ui.present` in the same release as
the view, and keep every command working without them.**

`ui.*` identifiers are absent from the manifest today by scope, not by accident
(`plugins/devtools/bitty-plugin.toml` "capabilities"). Adding three heads is a
consent event, not a code detail: any manifest change recomputes the manifest
hash, and added capabilities block automatic update and require a
permission-diff approval (R-016)
([Plugin Platform RFC](../specifications/plugin-platform-rfc.md) "Grant
lifecycle", `Update` row). The record accepts that consequence rather than
routing around it:

1. **The permission diff is exactly three heads.** `ui.rich` is required for
   every mount and update; `ui.overlay` is additionally required for the
   `overlay` slot; `ui.present` authorizes drawing the retained block without
   input capture
   ([bitty.ui reference](../sdk/reference/ui.md) "Capability"). No head implies
   another, and none of the three grants input authority.
2. **`ui.overlay.focus` is deliberately not requested.** It gates the single
   Core-owned transient input capture, and only a handle mounted into `overlay`
   can own it (`ui.md` "Capability"). A passive monitor must never take
   keystrokes from the terminal, so the view uses the presentation-only path and
   never calls `bitty.ui.overlay.acquire`.
3. **`debug.control` stays absent.** The view is still read-only; no write
   capability is added anywhere.
4. **`lazy.events` gains `overlay.dismissed`.** That is an observation kind, not
   a capability identifier, so it is not part of the permission diff; it is
   disclosed in the same note because it changes the manifest hash, and delivery
   requires the declaration ([bitty.ui reference](../sdk/reference/ui.md)
   "`overlay.dismissed` observation").
5. **Grant-less runs keep all seven existing commands intact.** They never touch
   `ui.*`, so nothing changes for a generation that has not re-consented. The
   five view commands return a bounded notice naming the missing head —
   `E_CAPABILITY_DENIED: capability not granted` through the existing
   `format.error` path — and never raise (`init.lua:58-80`, `:70-76`).
6. **The mount attempt must be guarded.** `init.lua` resolves
   `bitty.ui.mount` through the existing `bridge_fn` helper and calls it inside
   `pcall`, so an ungranted or older host degrades to the degraded run instead
   of failing activation. A plugin that fails activation because an optional
   surface was denied would be a regression for every existing user.

The version consequence: the view ships as a **minor** version (`0.2.0`), since
it adds commands and capabilities without changing the meaning of an existing
command.

## W-4 pause, resume, and mount lifecycle

**Decision: mount on first open, present to open, hide to close, keep the
handle across pause, stop it on close.**

The block cannot be destroyed, so the lifecycle is defined over presentation and
the view trace rather than over the block:

| State    | Block             | Surface   | View trace              | Timer   |
| -------- | ----------------- | --------- | ----------------------- | ------- |
| `closed` | mounted or absent | hidden    | closed                  | none    |
| `open`   | mounted           | presented | open, drained each tick | 1 armed |
| `paused` | mounted           | presented | open, **not drained**   | none    |

Transitions:

1. **`view-open`**: mount if no block exists, open the view trace
   (`max_events = 256`, filter per W-1 rule 5), `present.show(block)`, arm the
   first tick. A mount failure is terminal for the command and reported; a
   present failure (`E_UI_UNAVAILABLE` when another plugin holds the
   presentation slot, `E_CAPABILITY_DENIED` without `ui.present`) leaves the
   block mounted and the view closed.
2. **`view-pause`**: cancel the timer. The view trace stays open and is not
   drained, so records accumulate in a ring that is already bounded at 256
   records and 1 MiB; the drops that overflow it are counted and surface on
   resume. Pause therefore freezes the frame without leaking the handle and
   without pretending the stream stopped.
3. **`view-resume`**: arm the next tick. The first drain after a pause returns
   the accumulated window in one bounded batch and reports the accumulated
   drops, so the pause is visible in the loss counter rather than hidden.
4. **`view-close` / `Esc`**: cancel the timer, drain the final records once
   (counting drops), stop the view trace, `present.hide(block)`. The block
   stays mounted and is re-presented by the next `view-open`.
5. **Generation disposal** (reload, suspend-then-dispose, crash, rollback): the
   host drops the block, the timer, and the handle together, because all three
   are generation-owned
   ([bitty.timers reference](../sdk/reference/tasks-timers.md) "Capability";
   [bitty.ui reference](../sdk/reference/ui.md) "Returns"). The plugin does
   nothing on teardown and re-mounts on the next open.

Mounting on first open rather than at activation keeps a lazy plugin from
minting a retained block before the user asks for the view. That path is a
command callback calling `bitty.ui.mount`; the mount gate itself is
grant/claim/placement/registry only, with no activation-only check
(`crates/bitty-runtime/src/plugin_runtime/services.rs:5682-5736`,
`bitty@23ff0f6c`), but no shipped test currently mounts from a command
callback, so the verification plan pins it. If it cannot be pinned, the
documented fallback is mount at activation plus present on open — a strictly
smaller change to the same lifecycle table.

## W-5 dropped-record accounting

**Decision: the view accumulates a session total and shows the per-drain delta
alongside it, and the counter is always rendered.**

The host's `dropped` count is per drain, not cumulative: `trace_get` returns
`{ records, dropped }` and resets the counter after reporting
(`crates/bitty-lua/src/host.rs:2634-2640`; `TraceHub::drain` uses
`std::mem::take`). A view that printed the field directly would show `0` almost
every tick and hide exactly the loss it exists to expose. Rules:

1. `session_drops = session_drops + drain.dropped` on every drain, including
   the final drain on close.
2. The header always renders `dropped: <session_drops>`, even at zero. A counter
   that appears only on loss is a counter nobody trusts.
3. When a drain reports `dropped > 0`, the header appends
   `(+<n> this tick)`, and the trace section's summary line carries the same
   number so a paused-then-resumed burst is attributable.
4. The count is per handle. The command trace's drops stay in the command
   trace's output and are never merged into the view's total (W-1 rule 3).
5. The total resets when the view trace is opened, because a new handle has a
   new counter; the reset is announced in the open notice.

## Data shape

The view emits one `Column` root. Only the four accepted v1 kinds appear —
`Text`, `Row`, `Column`, `List`. `Image`, `CodeBlock`, `Table`, and `Rule` are
excluded from v1 and are **v2+**, so no table is drawn with a table node: the
"inspect tables" are `List`s of `Text` rows, exactly as the existing string
renderers already produce them
([bitty.ui reference](../sdk/reference/ui.md) "Nodes, styles, and typed
validation"; "Limits").

```text
Column
├─ Row                     header
│  ├─ Text "DevTools live"  (bold)
│  ├─ Text "running" | "paused"
│  ├─ Text "dropped: 12 (+3 this tick)"
│  └─ Text "1.0s"          (effective cadence)
├─ Column                  inspect section
│  ├─ Text "plugins (37, host list truncated)"
│  └─ List
│     └─ Text × ≤12        "id version [state] gen n"
├─ Column                  trace section
│  ├─ Text "trace 256 records, 12 dropped, filter terminal.*"
│  └─ List
│     └─ Text × ≤12        "#seq +12ms topic k=v k=v"
└─ Row                     footer hint
   └─ Text "p pause  r resume  t target  q close"
```

Rules for the tree:

1. **One inspect target per frame.** The view shows the selected target
   (`plugins`, `commands`, `events`, or `grants`, cycled by `view-target`), not
   all four, so a tick is one inspect call rather than four. `panels` stays
   unreachable: the host answers `E_NOT_IMPLEMENTED` for it
   (`crates/bitty-runtime/src/plugin_runtime/services.rs:7120-7125`), and
   `format.INSPECT_TARGETS` already omits it
   (`plugins/devtools/lua/devtools/format.lua:148-150`).
2. **Row text reuses the existing renderers.** `format.field` (64 bytes),
   `format.truncate` (112 bytes per line), and the `plugins` / `commands` /
   `events` / `grants` row shapes are reused unchanged, so a row in the view and
   a row in the notification body say the same thing
   (`format.lua:94-146`, `:127-146`).
3. **Twelve rows per list.** The body cap of 15 lines becomes 12 list rows plus
   two headers and one footer, and the overflow marker keeps the existing
   `(+N more)` / `(N earlier records omitted)` wording (`format.lua:178-180`,
   `:254-257`). The omitted counts are never dropped silently.
4. **Bounds.** The tree is ~40 nodes at depth 4 with ≤ 8 KiB of text, inside the
   stricter bridge budget (depth 16, 2048 nodes, 256 KiB text per block, 64
   retained blocks, 2 MiB retained string bytes per generation) and inside
   `SCN-1`..`SCN-5`
   ([bitty.ui reference](../sdk/reference/ui.md) "Per-generation bounds";
   [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)).
   A bound violation fails closed with `E_UI_COMPONENT_INVALID` at the bridge
   before any host call, so an over-wide row list can never reach the renderer.
5. **Presentation budget.** The presentation overlay admits at most 64 rows and
   16 KiB per frame and paints nothing in a frame narrower than 10 columns
   while keeping the session live
   ([bitty.ui reference](../sdk/reference/ui.md) "Admission and placement"), so
   a 27-row tree is inside the frame budget and a very narrow window shows an
   empty surface rather than a truncated one.
6. **No interactive nodes.** `on_click` / `on_scroll` are omitted entirely: on
   an overlay they are retained but not routed, and declaring a binding that can
   never fire would misrepresent the surface (`ui.md` "Band-only interaction
   routing and overlay capture"). The footer row is a text hint, not a control.

## Failure and denial modes

Every row is a bounded string in the view header or a bounded notice, never a
raise, matching the plugin's existing error discipline
(`plugins/devtools/lua/devtools/init.lua:58-80`).

| Condition                                                                                             | Observable outcome                                                                                                                                                                             |
| ----------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ui.rich` absent                                                                                      | `view-open` mounts nothing and returns `view unavailable: E_CAPABILITY_DENIED: capability not granted (ui.rich)`. The seven existing commands are unchanged.                                   |
| `ui.overlay` absent, `ui.rich` granted                                                                | Same path, naming `ui.overlay`; the host gates the `overlay` slot on it (`services.rs:5682-5700`).                                                                                             |
| `ui.present` absent                                                                                   | The block mounts, `present.show` fails with `E_CAPABILITY_DENIED` naming `ui.present`; the view stays closed and the retained block is counted against the 64-block budget.                    |
| Presentation slot held by another plugin                                                              | `present.show` fails with `E_UI_UNAVAILABLE`; the incumbent is untouched and the view stays closed ([bitty.ui reference](../sdk/reference/ui.md) "Second claimant").                           |
| Timer cap reached (`E_DEF_LIMIT` on create)                                                           | The view presents one static frame reading `refresh unavailable: E_DEF_LIMIT` and arms no chain; `view-resume` retries. No raise, no loop.                                                     |
| View trace unavailable (`E_DEF_LIMIT`, `E_CAPABILITY_DENIED`, `E_NOT_IMPLEMENTED`, `E_BRIDGE_ABSENT`) | The view runs: inspect tables refresh on the tick, the trace section reads `trace unavailable: <code>`, and the drop counter reads `dropped: -`.                                               |
| Host dropped the view trace (`trace_get` returns nil)                                                 | The session resets to `E_TRACE_GONE`, the view reopens the trace once on the next tick, and the header reports `trace restarted`; a second consecutive loss stops the chain with a notice.     |
| A bridge call fails inside a tick (`E_TIMEOUT`)                                                       | That tick is skipped, the previous frame stays, and the header reports `tick skipped: E_TIMEOUT` until the next successful tick clears it.                                                     |
| `Esc` dismissal (`overlay.dismissed`, `reason = "escape"`)                                            | The chain is cancelled, the final records drained and counted, the handle stopped, and the block hidden. The next `view-open` re-presents the same block.                                      |
| `reason = "view-closed"`                                                                              | Same as `Esc`; the terminal the panel was drawn over is gone.                                                                                                                                  |
| Plugin reload / generation disposal                                                                   | Block, timer, and handle are dropped with the generation; the surface disappears and the next open re-mounts. Nothing leaks across generations because all three handles are generation-owned. |
| Paused                                                                                                | Header reads `paused`, no timer is armed, and zero bridge calls are made per second. The ring keeps filling to 256 records and the overflow is counted.                                        |

## What this does not do

- **No new Core API.** No new bridge function, error code, capability
  identifier, event kind, slot, node kind, budget, or manifest schema field.
  `refresh_ms` is a plugin-local `[settings]` key inside the existing manifest
  schema, not a Core key. The
  [surface inventory](#why-a-gui-face-needs-no-new-core-api) is the evidence; a
  gap found during implementation becomes a Core issue with that evidence, not
  a change here.
- **No panel registration.** `register_panel` and `PanelProvider` remain
  excluded from plugin API v1; the view is an `overlay`-slot block, not a panel,
  and has no panel identity, activity stack, or focus participation.
- **No renderer handle and no per-frame Lua loop.** The plugin never draws; it
  hands the host a declarative tree on a timer and the host paints it. Nothing
  in this record runs on the input or render hot path.
- **No input authority.** `ui.overlay.focus` is not requested, `overlay.acquire`
  is never called, and the surface is paint-only.
- **No write capability.** `debug.control` stays uncalled and unrequested.
- **No raw PTY bytes and no cell-level inspection.** Redaction is unchanged: the
  trace backend redacts each record for the owner's own grants before it is
  stored (`debug.rs` module docs), and a GUI does not exempt that rule.
- **No v2 nodes.** `Image`, `CodeBlock`, `Table`, and `Rule` are v2+; the view
  uses `List` of `Text` instead.
- **No persistence or export.** Records live in a 256-record ring and are gone
  when the handle closes.
- **No second surface.** One view, one slot, one window. A `bottom`-band variant
  is not part of this decision (see [Open points](#open-points)).

## Security review

This record adds capability identifiers to a plugin's manifest and adds a
second trace handle and a periodic timer, so it is reviewable on the security
axis rather than exempt:

- **Capability widening is disclosed and consented.** Three `ui.*` heads are
  added and the update is blocked behind a permission diff (R-016). None of the
  three grants input, write, or cross-plugin read authority; `ui.overlay.focus`
  and `debug.control` stay unrequested.
- **No new trust boundary.** The view reads the same two surfaces the commands
  already read (`debug.inspect`, `debug.trace`) and writes only to a slot the
  host owns. Presentation never becomes Terminal Truth (invariant 3).
- **No hot-path execution.** The tick is a cold-tick timer pump; the surface
  receives no input; no plugin VM runs on the input or paint path (invariant 4).
- **Bounded inputs and outputs.** Every string reuses the existing byte bounds,
  the scene respects the per-generation and `SCN` bounds, the trace ring is
  capped at 256 records and 1 MiB, and every denial is a bounded code plus a
  hint.
- **Redaction is unchanged.** Trace payloads are redacted per recipient view
  before storage; the view cannot see more than a subscriber with the same
  grants.

The remaining risk is disclosure-by-rendering: a live view makes plugin
lifecycle data continuously visible where a command made it visible on demand.
That is a user-visible surface decision, not a new data path, and the data is
already available to the same plugin through the same calls.

## Verification plan

1. Mount from a command callback: `view-open` mounts the block and returns a
   positive `block_id`; if the host refuses that path, the fallback in
   [W-4](#w-4-pause-resume-and-mount-lifecycle) is implemented instead and this
   item is re-scoped.
2. The seven existing commands produce identical results with the view open and
   closed, including `trace-dump` draining only the command handle.
3. Two handles over one event: one delivered event appears in both drains;
   draining the view handle leaves the command handle's records and counter
   untouched, and vice versa.
4. A tick performs exactly three bridge calls (`inspect`, `trace_get`,
   `update`) and zero `notify` calls, asserted with a counting host double.
5. `view-pause` stops the chain (no calls over several cadences) and
   `view-resume` drains the accumulated window in one batch with the drop count
   it accumulated while paused.
6. `view-close` and `Esc` leave `trace_count == 0` for the view handle and
   release the timer slot; generation disposal leaves nothing behind.
7. The emitted tree contains only `Text` / `Row` / `Column` / `List`, stays
   under the per-generation node, depth, and text bounds, and fails closed with
   `E_UI_COMPONENT_INVALID` when a row list is grown past them.
8. With the three `ui.*` heads removed, all twelve commands return bounded
   notices, activation succeeds, and no mount is attempted.
9. Cadence clamping: `refresh_ms` below 250 renders `250ms`, above 60 000
   renders `60s`, and a non-integer or absent setting renders the 1000 ms
   default; the header always shows the effective value.
10. Update-while-presented repaints without a second `present.show`, and a
    `present.show` after `Esc` re-presents the same block.

## Alternatives considered

| Alternative                                       | Trade-off                                                                                    | Disposition                                                                     |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Live view owns the shared trace handle while open | One handle, one drain, simplest state machine                                                | Rejected — makes `trace-dump` / `trace-stop` depend on whether a window is open |
| Second handle, commands untouched                 | One extra copy per matching event; two rings to bound                                        | **Accepted** (W-1)                                                              |
| `bottom`-band surface instead of `overlay`        | Clickable pause/resume through band-only routing; permanent layout height; no way to unmount | Rejected for v1 — a monitor should not permanently shrink the terminal          |
| `overlay` + `overlay.acquire` capture session     | Routed input on the panel                                                                    | Rejected — steals keystrokes from the terminal for a read-only monitor          |
| Repeating timer                                   | Simpler than a re-armed chain                                                                | Not available in v1; one-shot chain is the accepted idiom                       |
| Refresh at 100 ms for a "live" feel               | Smoother stream                                                                              | Rejected — 10× the drain and encode cost for data that changes per second       |
| Mount at activation                               | Uses only the proven mount path                                                              | Fallback only, if mount-from-command cannot be pinned                           |
| Print the raw per-drain `dropped`                 | No accumulation logic                                                                        | Rejected — hides loss, which is the point of the counter                        |
| A new Core API for a "live view" surface          | Would allow push-based rendering                                                             | Rejected — the shipped timer + update pair already expresses it                 |

## Affected contracts

| Contract                                                                            | Effect                                                                                                       |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted) | Unchanged; the record consumes only its spellings and cites its caps.                                        |
| [bitty.ui reference](../sdk/reference/ui.md) (draft)                                | Gains a worked consumer of `mount` / `update` / `present`; states no new host behavior.                      |
| [bitty.timers reference](../sdk/reference/tasks-timers.md) (draft)                  | Gains a worked re-arming chain; states no new host behavior.                                                 |
| `bitty-featured.devtools` manifest (`bitty-plugins`)                                | Adds `ui.rich`, `ui.overlay`, `ui.present`, five commands, `[settings].refresh_ms`, and `overlay.dismissed`. |
| `bitty-featured.devtools` trace session (`trace.lua`)                               | Gains a second, view-owned handle slot; the command handle's semantics are unchanged.                        |
| [Plugin UI Slot Inventory](plugin-ui-slot-inventory-candidate.md) (candidate)       | Supplies the `overlay` multiplicity and bounds this record stays inside.                                     |
| [Plugin UI Component Catalog](ui-component-catalog-candidate.md) (candidate)        | Its status-strip and detail recipes cover the header and list shapes used here.                              |

## Open points

These need an owner decision; this record does not settle them.

- **Blur pause.** v1 has no window-focus or visibility observation, so "pause
  when the window loses focus" is not expressible. Owner decision: accept
  "not presented" as the definition of hidden, or file a Core gap for a
  window-focus observation kind.
- **View-trace record bound.** 256 records is this record's recommendation, not
  a host constant. The trace budget owner confirms it, or names the value the
  view must use instead of the 1000-record host default.
- **Where the cadence setting lives.** A manifest `[settings]` `int` is the
  mechanism; whether the value is edited in the plugin-manager UI, the user
  configuration file, or both is the plugin-manager owner's call.
- **Keymap binding for pause/resume.** Adding `bitty.keymaps` registration to
  this plugin is out of scope here and would need its own grant and UX review.
- **Band fallback.** Whether a `bottom`-band variant should exist as the
  degraded surface when `overlay` is unavailable, instead of notifications, is
  a product decision. This record keeps notifications as the fallback.
- **Consent copy.** The plain-language effect statements for the three `ui.*`
  heads are owned by the consent UX, not by this document.
- **Multi-view.** Whether more than one DevTools view may exist per generation
  (for example a trace-only view beside an inspect-only view) is deferred; this
  record fixes one surface and two trace handles.

## Acceptance criteria

1. A reviewer confirms no Core API, error code, capability identifier, event
   kind, node kind, or budget is added by this record.
2. Each of W-1 through W-5 states one decision, its failure mode, and why the
   rejected option was rejected.
3. The refresh budget names its default, its configuration path, its floor, and
   every ceiling it must stay inside, each cited to the owning source.
4. The grant decision states the consent consequence and the grant-less
   behaviour of every existing command.
5. The data shape uses only the four accepted v1 kinds and states the bounds it
   respects.
6. Every failure mode has an observable outcome and none raises.
7. Open points are listed as owner decisions rather than silently decided.
8. Implementation proceeds in the plugin repository against this record; a
   change to a decision here is a new revision of this record, not a code
   review comment.

## P0 Review Sign-off

Required before implementation: the change adds three capability heads to a
DevTools plugin's manifest, holds a second trace handle, and arms a periodic
timer. The security review above records the disposition (capability widening is
consented and disclosed, no input authority, no hot-path execution, bounds and
redaction unchanged). Sign-off owners: the plugin-security reviewer for the
capability diff, the trace budget owner for the view-trace bound, and the
extensibility category owner for the contract text.

## References

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) —
  accepted slot set, node subset, handle model, `RC-4` caps, one-shot timers,
  closed event names.
- [bitty.ui reference](../sdk/reference/ui.md) — hosted slots, node and style
  validation, per-generation bounds, `ui.mount` gate order, presentation-only
  overlay, `overlay.dismissed`, error vocabulary.
- [bitty.tasks and bitty.timers reference](../sdk/reference/tasks-timers.md) —
  timer capture bounds, delay ceiling, one-shot semantics, `E_DEF_LIMIT`.
- [bitty.events reference](../sdk/reference/events.md) — closed kind set,
  declared-kind precondition, activation-only subscription.
- [bitty.settings reference](../sdk/reference/settings.md) —
  `bitty.settings.get` over a manifest declaration.
- [bitty.notify reference](../sdk/reference/notify.md) — acceptance boolean and
  host-side rate policy.
- [Plugin Platform RFC](../specifications/plugin-platform-rfc.md) — grants,
  grant lifecycle and the R-016 permission diff, three-level queue budgets.
- [Isolation Resource RFC](../runtime/isolation-resource-rfc.md) — `RC-4`.
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  — `SCN-1`..`SCN-5`.
- [DevTools RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/devtools-rfc.md)
  — debug scopes and the observability pipeline (the protocol side of tracing).
- [Plugin UI Slot Inventory (Candidate)](plugin-ui-slot-inventory-candidate.md) —
  slot multiplicity and bounds.
- [Plugin UI Component Catalog (Candidate)](ui-component-catalog-candidate.md) —
  composition recipes over the v1 nodes.
- `bitty` at `23ff0f6c`:
  `crates/bitty-runtime/src/plugin_runtime/debug.rs` (trace ring, caps, drain
  counter, `parse_trace_opts`), `crates/bitty-lua/src/host.rs` (`ui` table,
  `debug_trace_get` contract), `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (`ui_mount` gates), `crates/bitty-runtime/src/plugin_runtime/mod.rs` (timer
  deadlines and the cold-tick pump), `crates/bitty-runtime/tests/plugin_timers.rs`
  (re-arm chains, slot reuse).
- `bitty-plugins` at `2da1c87`: `plugins/devtools/lua/devtools/init.lua`,
  `trace.lua`, `format.lua`, and `plugins/devtools/bitty-plugin.toml`.
