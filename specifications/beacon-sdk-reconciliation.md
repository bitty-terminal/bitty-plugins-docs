---
title: Beacon Contract and SDK Reconciliation
description: Draft reconciliation mapping every candidate Beacon policy operation to an accepted Plugin API v1 surface or naming the task that blocks it
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 29
---

# Beacon Contract and SDK Reconciliation

> Status: **draft** reconciliation. It maps the candidate Beacon policy contract
> in `docs/plugins/beacon/` to the accepted Plugin API v1 Lua surface and to the
> owner-pending overlay and input-capture surfaces. It accepts no new SDK
> method, capability, quota, or event; it decides no `OQ-056` or `OQ-088`
> content; and it makes no implementation claim. Every **Accepted** row below
> cites the accepted contract that authorizes it; every **Blocked** row names
> the task that blocks it. This page is the SDK-owner (`W-91`) view and does not
> restate or weaken the Beacon policy contract or the terminal-side mechanism.

## Document status

This page reconciles two candidate documents with the accepted surface that
actually exists today. The policy side is the Beacon plugin page set
([README](../docs/plugins/beacon/README.md), [design](../docs/plugins/beacon/design.md),
[schemas](../docs/plugins/beacon/schemas.md), [evidence](../docs/plugins/beacon/evidence.md)),
which is candidate direction: no `bitty-terminal/beacon` repository, package,
manifest, or VM exists. The mechanism side is the terminal-side
[Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md)
(`W-81`), whose `TargetEngine`/`AnnotationEngine` names and extraction scope are
accepted by
[ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md),
while its host API is candidate (`W-29`). The overlay and input-capture host API
the Beacon key language depends on is owner-pending (`W-01`, Issue
[bitty-docs#396](https://github.com/bitty-terminal/bitty-docs/issues/396), under
`OQ-056`), and its terminal-side shape is only provisional in the accepted
[Composer Architecture and Host API](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md)
(`W-82`).

The reconciliation result is that Beacon's policy **declaration** operations
(chords, commands, settings, persistence, observation of accepted events) map to
accepted v1 surfaces today, while every Beacon operation that needs the target
mechanism, annotation presentation, a focusable overlay, or transient input
capture is explicitly blocked and named. No "blocked" row is a promise that the
surface will exist; it names its blocking task.

- Owning task: `W-91` (bitty-plugins-docs), CarryCtx `CTX-0071`, Issue
  [bitty-plugins-docs#129](https://github.com/bitty-terminal/bitty-plugins-docs/issues/129).
- Reconciles: the `W-90` page set (`CTX-0069`, Issue #128) against the accepted
  [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) and
  the owner-pending `W-01`/`W-28`/`W-29` surfaces.
- Depends on: `W-01`, `W-28`, `W-43`, `W-90` (cross-repository).

## Purpose and scope

The purpose is to state, without inventing surface, exactly which accepted SDK
contract each Beacon policy operation stands on, which operations are blocked,
and which existing or named task supplies the missing method, capability, quota,
or event. This prevents the candidate page set from implying an API that the
accepted v1 surface and the owner-pending host contract do not provide.

In scope:

- an operation-by-operation mapping of the Beacon policy contract to a concrete
  surface: an accepted SDK contract named with its exact spelling, a
  candidate/provisional surface with its owner, or an explicit block with the
  blocking task named;
- the missing SDK methods, capability declarations, quotas, and
  event/dispatch semantics, and the incompatible assumptions between the Beacon
  policy contract and the accepted v1 and provisional overlay surfaces;
- the no-hidden-privilege statement for the Beacon plugin;
- the fail-closed negative-path behavior for rejected, unauthorized, and
  unsupported operations;
- the named follow-up SDK, host-API, and Core tasks, each with an owner.

Out of scope and owned elsewhere:

- the exact public host names, capability identifiers and dimensions, and API
  version for the target mechanism (`W-29`) and the focusable overlay and
  input-capture mechanism (`W-01`);
- the SDK content that will carry those names (`W-120`/`CTX-0065`, `W-43`,
  and the history/search/selection surface at `W-139`/`CTX-0066`);
- the Beacon plugin package, manifest, and implementation (`W-53`, `W-121`);
- the Core extraction and policy retirement (`W-102`, `W-30`);
- the open questions `OQ-056`, `OQ-052`, and `OQ-088`, which stay open with
  their owners.

## Normative sources this specification must not weaken

This reconciliation must be read together with, and must not weaken:

- The accepted [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (ADR 0009): the frozen v1 Lua function surface, payloads, event-name set, and
  the explicit exclusions. This page may not add, rename, or reinterpret a v1
  identifier.
- The accepted [Plugin Platform RFC](plugin-platform-rfc.md): the manifest
  schema, the closed capability-identifier families, deny-by-default grant
  lifecycle, the command registry, and the event pipeline.
- The accepted [Plugin Manifest and Capability Grammar Authority](manifest-capability-authority.md):
  the canonical manifest spelling and capability grammar.
- The accepted [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md):
  per-plugin VM isolation and resource ceilings.
- The accepted [Composer Architecture and Host API](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md)
  (`W-82`): the provisional focusable-overlay, transient input-capture, and
  bounded submission host-API shape, which is provisional until `W-01` accepts
  its contract.
- The terminal-side [Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md)
  (`W-81`): the accepted mechanism/policy split, the accepted mechanism names,
  and the retained fail-closed fences.
- [ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md)
  and [ADR 0015](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0015-small-core-extraction-boundaries.md)
  (Boundary 3 and the bootstrap fence): the accepted split and the prohibition
  on a private first-party bypass, raw PTY/GPU/window handles, and input
  hot-path callbacks.

If any statement on this page contradicts a normative source, the normative
text wins. The [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
is the authority for accepted v1 spellings; this page quotes, never defines,
them.

## Terminology

| Term                            | Meaning in this document                                                                                                                                                     |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Accepted surface                | A Lua function, capability identifier, quota, or event name fixed by an accepted contract (ADR 0009 or the Plugin Platform RFC).                                             |
| Provisional surface             | A spelling stated as a proposal or as a wired host candidate pending acceptance (`W-01`, `OQ-056`); no implementation may cite it as contract.                               |
| Blocked                         | No accepted or provisional surface exists; the row names the task that must supply it before any implementation.                                                             |
| Beacon policy operation         | One operation the candidate Beacon design claims: key language/keymaps, which-key/menu, scopes/filters, theme badges, provider composition, target-first menus, or dispatch. |
| Accepted v1 event set           | The closed `EventKind` name set in the Plugin API v1 Lua Surface RFC; no other event name is expressible in v1.                                                              |
| Transient input capture         | The Core-owned, capability-gated mechanism that takes exclusive keyboard and overlay focus for one bounded interaction; it has no v1 Lua surface and is under `OQ-056`.      |
| `W-91` operation-to-surface map | The table in [Operation-to-surface reconciliation](#operation-to-surface-reconciliation) that resolves each Beacon policy operation.                                         |

## Accepted surface baseline

The accepted v1 surface is the only SDK contract the Beacon plugin may rely on
today. It is fixed by the [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
and mirrored, as parity evidence rather than normative text, in the generated
SDK surface
`bitty-plugin-sdk/surface/bitty-plugin-api-v1.json`. The functions relevant to
Beacon policy are:

| Accepted v1 function                                              | Relevance to Beacon policy                                                 |
| ----------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `bitty.keymaps.suggest(def) -> handle`                            | Declares chord-to-command suggestions; never forces input mutation.        |
| `bitty.commands.register(def) -> handle`                          | Registers the typed operator, menu, and context commands.                  |
| `bitty.events.subscribe(name, handler) -> handle`                 | Observes the accepted closed v1 event set only.                            |
| `bitty.ui.mount(slot, component) -> handle`                       | Contributes bounded declarative content to an accepted slot.               |
| `bitty.ui.update(handle, component) -> boolean`                   | Replaces mounted content under the same block handle.                      |
| `bitty.settings.get(key)` / `bitty.settings.set(key, value)`      | Reads and writes policy configuration under `plugins.<id>`.                |
| `bitty.store.get(key)` / `bitty.store.set(key, value)`            | Persists quota-bounded policy state.                                       |
| `bitty.notify.show(payload) -> boolean`                           | Emits a bounded notification under `platform.notify`.                      |
| `bitty.terminal.snapshot(opts) -> Snapshot`                       | Reads the accepted semantic snapshot only, under `terminal.semantic-read`. |
| `bitty.services.get(iface, opts)` / `bitty.services.provide(...)` | Resolves a versioned plain-value service; no cross-VM handle.              |
| `bitty.tasks.spawn/cancel`, `bitty.timers.create/cancel`          | Runs bounded host-owned asynchronous work.                                 |

The accepted capability families that Beacon policy may use without a new
contract are also closed: `ui.rich`, `ui.overlay`, `terminal.semantic-read`,
`platform.notify`, and `env.read:<KEY>`. The accepted v1 surface contains
**no** target, annotation, provider-registration, label, session,
focusable-overlay, or transient input-capture function or capability.
`ui.overlay` is presentation-only and non-focusable, and the `overlay` slot
fails closed with `E_UI_UNAVAILABLE` in the current host build.

The generated parity surface additionally carries `bitty.workspace.*`
functions and workspace observation events, wired in `bitty` under ADR 0014,
plus `workspace.read`/`workspace.control` capability spellings. Those spellings
are explicitly **host candidates pending `OQ-056`** in the parity artifact's
own note, and the accepted Plugin Platform RFC capability families do not
include a `workspace` family; this page therefore treats them as provisional,
not as accepted v1 contract.

## Operation-to-surface reconciliation

Each row resolves one candidate Beacon policy operation. **Accepted** cites the
accepted contract; **Provisional** cites the owner-pending proposal and states
that no implementation may cite it; **Blocked** names the task that must supply
the surface.

| Beacon policy operation                                                            | Concrete surface (exact spelling or owner)                                                                                                   | Status                                               | Source or blocking task                                  |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- | -------------------------------------------------------- |
| Declare a chord-to-operator suggestion                                             | `bitty.keymaps.suggest(def)`, precedence user > workspace > first-party/default > plugin suggestion                                          | Accepted                                             | Lua Surface RFC; Plugin Platform RFC                     |
| Register the typed command each operator or menu entry executes                    | `bitty.commands.register(def)`                                                                                                               | Accepted                                             | Lua Surface RFC; Plugin Platform RFC                     |
| Persist operator families, default scopes, filters, badges, and menu order         | `bitty.settings.get/set`, `bitty.store.get/set`                                                                                              | Accepted                                             | Lua Surface RFC; Isolation and Resource RFC              |
| Observe focus, selection, terminal, and process state that policy reads            | `bitty.events.subscribe` over accepted v1 names (`focus.changed`, `selection.changed`, `process.exited`, `config.reloaded`, `terminal.*`)    | Accepted                                             | Lua Surface RFC; Plugin Platform RFC                     |
| Observe workspace lifecycle or state for a workspace scope                         | `workspace.*` observation events                                                                                                             | Provisional (host candidate, `OQ-056`)               | Generated parity surface; `OQ-056`                       |
| Focus or move to a workspace as a policy action                                    | `bitty.workspace.focus` / `bitty.workspace.move_panel`                                                                                       | Provisional (host candidate, `OQ-056`)               | Generated parity surface; `OQ-056`                       |
| Emit a bounded policy notification                                                 | `bitty.notify.show(payload)`                                                                                                                 | Accepted                                             | Lua Surface RFC; Plugin Platform RFC                     |
| Present bounded static which-key hint or menu text                                 | `bitty.ui.mount(slot, component)` into an accepted band slot (`top`, `bottom`, `statusline` painted; `left`/`right` stored, not yet painted) | Accepted (function and slots); payload shape `W-120` | Lua Surface RFC; payload shape `W-120`                   |
| Present an interactive which-key continuation or focusable menu                    | Focusable interactive menu surface (no accepted spelling)                                                                                    | Blocked                                              | `W-28` host implementation; `W-120` SDK                  |
| Start a Beacon session that captures keys                                          | Focusable overlay plus transient input capture (`bitty.overlay.acquire/poll/release`, `ui.overlay.focus`)                                    | Blocked                                              | `W-01` contract; `W-28` host implementation; `W-120` SDK |
| Enter a target-first or operator-pending interaction and label targets             | Beacon Lua host API over `TargetEngine`/`AnnotationEngine` (no accepted spelling)                                                            | Blocked                                              | `W-29` host API; `W-120`/`W-43` SDK                      |
| Collect, scope, and filter the candidate target set                                | Target snapshot, provider collection, and scope resolution (no accepted spelling)                                                            | Blocked                                              | `W-29` host API; `W-120`/`W-43` SDK                      |
| Compose which registered providers and target kinds an operator or scope considers | Provider registration and tier composition (no accepted spelling or capability)                                                              | Blocked                                              | `W-29` host API; `W-120`/`W-43` SDK; `OQ-056`            |
| Contribute badge metadata mapped to a theme token                                  | Annotation-layer contribution (no accepted spelling; `bitty.annotations` is excluded from v1)                                                | Blocked                                              | `W-29` host API; `W-120`/`W-43` SDK                      |
| Build and drive a target-first context menu from a target's declared actions       | Target metadata plus a focusable interactive menu (no accepted spelling)                                                                     | Blocked                                              | `W-29` host API; `W-28` host implementation; `W-120` SDK |
| Resolve a selected target or menu entry to a typed command id and dispatch it      | Command registry execution is accepted; the target-to-command dispatch bridge is not (`W-29`)                                                | Blocked (bridge) / Accepted (execution)              | `W-29` host API; `W-120`/`W-43` SDK                      |
| Observe target or annotation events within a session                               | No target event exists; the closed v1 event set excludes them                                                                                | Blocked                                              | `W-29` host API; `W-120`/`W-43` SDK                      |

Reading of the map:

- **Accepted today:** chord suggestions, command registration, policy
  configuration and persistence, observation of the accepted event set,
  notifications, and bounded static content in the accepted band slots
  (`top`, `bottom`, `statusline`). These are enough to declare policy and to
  present non-interactive hint text; they are not enough to run a targeting
  session.
- **Provisional:** the `bitty.workspace.*` functions and workspace events whose
  Lua spellings are wired host candidates pending `OQ-056`. The target-first
  menu and the interactive which-key continuation surface remain blocked until
  the focusable overlay exists.
- **Blocked:** every operation that needs the target mechanism, the annotation
  layer, provider registration, a focusable overlay, transient input capture, or
  target events. The blocking tasks (`W-01`, `W-28`, `W-29`, `W-120`, `W-43`)
  are named in the table and in [Follow-up tasks](#follow-up-tasks).

## Missing SDK methods, capabilities, quotas, and event semantics

### Missing methods

The accepted v1 surface has no Beacon-relevant function for any of the
following; each must be added by `W-120`/`CTX-0065` (and, for the overlay and
input-capture portion, `W-43`) only after `W-01` and `W-29` accept their
contracts. No spelling is invented here.

- focusable-overlay acquire, update, poll/receive, and release;
- transient input-capture session start, cancel, and release;
- target registry snapshot and target metadata read;
- provider registration and provider selection/composition;
- label allocation request and label-to-command binding;
- annotation/badge contribution;
- command-dispatch bridge binding a target handle to a typed command id;
- target or session lifecycle events.

The accepted-slot presentation path (`bitty.ui.mount`/`update`) exists, but the
`overlay` slot is presentation-only, non-focusable, and currently unhosted
(`E_UI_UNAVAILABLE`); it cannot stand in for an interactive session.

### Missing capability declarations

The accepted capability families are closed and contain no provider
registration, target observation, or transient input-capture identifier; the
Plugin Platform RFC forbids plugins from inventing families and rejects any
allow-all identifier. The missing dimensions and their version are open under
`OQ-056` and owned by `W-29`:

| Needed capability dimension                     | Status                                                                                                       |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Target-provider registration                    | Missing; identifier and dimensions open (`OQ-056`, `W-29`).                                                  |
| Target metadata observation                     | Missing; default remains no cross-plugin observation (`W-29`, mechanism contract).                           |
| Transient input capture for one bounded session | Proposed as `ui.overlay.focus`, distinct from v1 `ui.overlay`; provisional until `W-01` (Composer contract). |
| Focusable overlay acquisition                   | Coupled to the capture capability above; owner `W-01`.                                                       |

Beacon requests no allow-all identifier and relies on deny-by-default; an
unknown identifier fails manifest validation.

### Missing quotas

Accepted quotas that apply to the accepted surfaces are the store ceiling
(`STORE_QUOTA_BYTES`, 256 KiB), the semantic-snapshot ceiling
(`SNAPSHOT_MAX_BYTES`, 256 KiB), the RC-4 task/timer caps (64 tasks / 32
timers), the command-schema ceiling (`CMD_SCHEMA_MAX_BYTES`, 16 KiB), and the
bounded event size. The Beacon mechanism budgets are **not** accepted: the
targets-per-kind (1024), targets-per-snapshot (1024), providers-per-mediator
(64), provider-name-length (32), annotations-per-session (1024),
labels-per-allocation (1024), label-charset-length (64), and
bindings-per-dispatcher (1024) values are implemented-only evidence in the
`bitty-ui` beacon artifacts, and the accepted ceilings are owned by `W-29`.
Beacon policy may not publish or depend on those numbers as contract.

### Missing event and dispatch semantics

- The closed v1 event set carries no target, annotation, label, session,
  provider, or input-capture event. Target observation within a session is
  therefore not expressible in v1 (`W-29`, `W-120`).
- The accepted `intercept.command-dispatch` event is a bounded veto/approve
  interception, not a dispatch trigger; it cannot bind a target to a command or
  start a dispatch.
- The command registry executes a registered command under its owner's grants;
  the target-to-typed-command bridge that revalidates a handle before dispatch
  is Core-owned and unexposed (`W-29`). Beacon may register and own commands,
  but it cannot resolve a selected target to a command id through v1.
- Hot-path events do not exist by design; no byte-received, cell-changed,
  damage, or per-frame event is expressible, so a plugin callback can never run
  on the input hot path.

## Incompatible assumptions

The candidate Beacon policy contract and the accepted v1 surface disagree on
five points. None is resolved here; each is parked to its owner.

1. **Fail-open transient capture.** The [design](../docs/plugins/beacon/design.md)
   assumes a session takes transient, bounded, Core-owned input capture whose
   non-label keys fail open to normal input. v1 has no capture surface, no
   input events, and only a presentation-only `overlay` slot that is currently
   unhosted. The assumption is unimplementable on v1 and needs `W-01`
   (contract), `W-28` (host implementation), and `W-120` (SDK binding).
2. **Plugin-contributed annotation.** The design assumes Beacon supplies public
   badge metadata and Core renders it through the `AnnotationEngine`. v1
   explicitly excludes Level 3 presentation (`bitty.annotations`,
   `bitty.decorations`, and `bitty.highlighting` are v1 exclusions), and the
   accepted `ui.mount` contract offers no annotation channel. The assumption is
   blocked pending `W-29`/`W-120`.
3. **Public provider registration and target observation.** The design assumes
   the plugin registers providers and observes public target metadata through
   the public API. v1 has no such identifier or function, and the capability
   families are closed with no allow-all form. The assumption is blocked under
   `OQ-056` and `W-29`.
4. **Interactive target-first menu.** The design assumes a menu built from a
   target's declared actions whose selection resolves to a typed command id.
   The accepted `overlay` slot is non-focusable and unhosted, offers no
   selection event, and gives no target metadata, so the interactive menu is
   blocked pending `W-28` and `W-120`.
5. **Dispatch-bridge reachability.** The design correctly assumes the selected
   target becomes a typed command that executes through the command registry.
   v1 accepts the registry execution half but exposes no target-to-command
   bridge, so the bridge half is blocked pending `W-29`/`W-120`. This is a
   partial overlap, not a conflict: the accepted command registry is the right
   execution path, and Beacon forwards no authority through it.

Two assumptions are **consistent** with the accepted surface and introduce no
conflict: chord declaration through `bitty.keymaps.suggest` matches the accepted
spelling and precedence, and policy configuration under
`plugins."bitty-terminal.beacon"` matches the accepted `plugins.<owner>.<name>`
settings namespace. The exact key surface still belongs to the plugin contract
and `W-120`.

## No hidden privilege

The Beacon plugin uses only public, capability-gated surfaces. Consistent with
the accepted fences in
[ADR 0015](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0015-small-core-extraction-boundaries.md)
and
[ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md),
this reconciliation records:

- no private first-party bypass: an equivalent third-party plugin reaches the
  same public surfaces, and there is no Beacon-only channel;
- no raw PTY, GPU, or window handle, and no raw PTY access, input injection,
  grid mutation, or cross-plugin VM access;
- no input hot-path callback: capture is Core-owned, transient, bounded,
  revocable, and fail-open, and never runs a plugin callback on the input hot
  path;
- no Event-Bus exposure of target or annotation internals, and no cross-plugin
  target-metadata observation by default;
- no direct call into the compositor, terminal, or panel lifecycle; a selected
  target becomes a typed command in the accepted registry and nothing else;
- registration grants no authority: contributing a provider or target adds
  addressability only, and a command executes under its own owner's grants with
  its own validation and budgets.

No row marked **Blocked** creates an exception; a blocked operation stays
blocked rather than descending to a private path.

## Negative-path behavior

Every rejected, unauthorized, or unsupported operation fails closed before any
side effect, following the accepted error vocabulary or the mechanism's typed
failure. The required behaviors are:

| Operation or condition                        | Required fail-closed behavior                                                                                      | Source                                    |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| Missing capability at a gated call            | Typed `E_CAPABILITY_DENIED` before any side effect; no provider, session, or overlay is created.                   | Lua Surface RFC; ADR 0018                 |
| Unknown or undeclared capability identifier   | Manifest validation rejects it; no allow-all identifier exists.                                                    | Plugin Platform RFC                       |
| Undeclared or unknown event name              | Registration error; the closed v1 event set is enforced.                                                           | Lua Surface RFC                           |
| Unknown or invalid chord in a suggestion      | Registration error naming the supported context; suggestions never override user or workspace mappings.            | Lua Surface RFC                           |
| Capability absent at provider registration    | Registration fails closed with a capability-denied error; no provider is registered.                               | Mechanism contract                        |
| Stale or unknown target handle, expired label | Fail closed with `StaleTarget`/unknown-label error; never a default target or a silent dispatch.                   | Mechanism contract                        |
| Oversized target snapshot or allocation       | Fail before any registry insert or allocation; no partial snapshot and no silent truncation.                       | Mechanism contract                        |
| Provider fault during collection              | The faulty provider's targets are absent and the session continues with the remaining providers.                   | Mechanism contract                        |
| Plugin crash, unload, or cancellation         | The session ends, capture is revoked, and labels are cleared; no dangling handle or orphaned layer survives.       | Mechanism contract                        |
| Absent, disabled, or safe-mode plugin         | Policy is absent; the Core mechanism works with zero plugins and `bitty --safe` starts with no third-party plugin. | ADR 0018                                  |
| Unsupported or explicit non-semantic snapshot | Rejected in v1; only the semantic scope is accepted.                                                               | Lua Surface RFC                           |
| Unhosted or capability-locked UI slot         | Typed `E_UI_UNAVAILABLE` or `E_CAPABILITY_DENIED`; no partial mount.                                               | UI reference                              |
| Oversized or invalid store value              | Typed `E_STORE_QUOTA` or `E_STORE_VALUE_INVALID`; no partial write or eviction.                                    | Lua Surface RFC                           |
| Task or timer over the RC-4 cap               | Typed `E_BUDGET_TASK` or `E_BUDGET_TIMER`; nothing is queued silently.                                             | Lua Surface RFC                           |
| Unimplemented optional surface                | Typed `E_NOT_IMPLEMENTED`; never a silent success.                                                                 | Host parity artifact (host error surface) |

No failure case may fall back to a default target, a truncated snapshot, a
silent dispatch, or a privileged path.

## Follow-up tasks

The following tasks are named as owners. This page does not decide their
content.

### SDK tasks

| Task                             | Owner            | Scope (name only)                                                               | Blocked on                |
| -------------------------------- | ---------------- | ------------------------------------------------------------------------------- | ------------------------- |
| `W-120` / `CTX-0065`, Issue #137 | bitty-plugin-sdk | Add only the accepted Beacon, overlay, and transient input-capture surface.     | `W-01`, `W-90`, `W-91`    |
| `W-43`                           | bitty-plugin-sdk | Overlay/input-capture and Beacon namespaces in the surface; re-pin host parity. | `W-28`, `W-29`            |
| `W-139` / `CTX-0066`, Issue #138 | bitty-plugin-sdk | Accepted history/storage/search/selection public APIs, mock host, conformance.  | `W-135`, `W-137`, `W-138` |

`W-139` is named because its selection and observation surfaces are adjacent to
the target session and event plumbing Beacon will need; this page does not fold
Beacon content into it.

`W-43` is the SDK-side companion of the host-API tasks `W-28` and `W-29`: it
adds the overlay/input-capture and Beacon namespaces to the SDK surface and
re-pins host parity once those host APIs land. It is owned by
`bitty-plugin-sdk`, not by the `bitty` host tasks it depends on.

### Host-API and contract tasks

| Task               | Owner        | Scope (name only)                                                           | Blocked on             |
| ------------------ | ------------ | --------------------------------------------------------------------------- | ---------------------- |
| `W-01`, Issue #396 | bitty-docs   | Focusable overlay and transient input-capture host API decision (`OQ-056`). | —                      |
| `W-28`             | bitty (Core) | Implement the focusable overlay / input-capture host API.                   | `W-01`                 |
| `W-29`             | bitty (Core) | Beacon Lua host API over the Core mechanism.                                | `W-28`, `W-03`, `W-12` |

### Core and plugin tasks

| Task                 | Owner         | Scope (name only)                                              | Blocked on             |
| -------------------- | ------------- | -------------------------------------------------------------- | ---------------------- |
| `W-102`              | bitty (Core)  | Extract the Beacon mechanism while retaining target safety.    | `W-01`, `W-81`, `W-90` |
| `W-30`               | bitty (Core)  | Remove Beacon policy from Core after the mechanism is stable.  | `W-29`, `W-53`         |
| `W-121` / `CTX-0030` | bitty-plugins | Create and register `bitty-terminal/beacon`.                   | `W-90`, `W-120`        |
| `W-53`               | bitty-plugins | Implement the Beacon policy plugin against the public surface. | `W-52`, `W-29`, `W-43` |

Ordering (recorded, not decided here): owner decision (`W-01`, `W-03`) -> host
contract (`W-81`, `W-29`) -> SDK contract and conformance (`W-120`, `W-43`,
`W-139`) -> host implementation (`W-28`) -> plugin implementation (`W-53`,
`W-121`) -> Core policy retirement (`W-30`, `W-102`) -> integrated verification.
No blocked row above may proceed by inventing a surface, and no SDK or Core task
content is fixed by this page.

## Security review

Beacon crosses the plugin trust boundary, the capability model, safe mode, and
the input and dispatch paths. This reconciliation changes no control and adds no
authority; independent security review is required before any promotion beyond
draft. The reviewer must confirm that:

- the Core mechanism stays capability-independent and always available, and
  safe mode keeps zero third-party plugins with the mechanism functional;
- the plugin uses only the public capability-gated API, with no private
  first-party bypass and no raw PTY, GPU, or window handle;
- transient input capture stays capability-gated, transient, bounded,
  revocable, and Core-owned, with no plugin callback on the input hot path;
- the command-dispatch bridge routes through the accepted command registry and
  revalidates handles before returning a typed command;
- target and annotation internals are not exposed through the Event Bus, and no
  plugin receives another's target metadata beyond the accepted scope;
- every blocked operation remains blocked rather than descending to a private
  path, and fail-closed behavior holds on denied capability, invalid label,
  stale handle, malformed metadata, and provider fault.

No P0 control is moved into the optional plugin by this page. The open points
below are parked, not waived.

## Verification plan

This is a reconciliation document; it has no executable verification of its own.
Any later implementation must produce, at minimum:

1. a capability-gating test proving a denied grant fails closed before any side
   effect, for the target-provider, observation, and input-capture dimensions;
2. a parity test proving an equivalent third-party plugin reaches the same
   public surfaces with no private bypass;
3. a safe-mode test proving targeting works with zero plugins and in
   `bitty --safe`;
4. a no-hot-path test proving capture is transient and revocable and no plugin
   callback runs on the input hot path;
5. a fail-closed test proving an invalidated target cannot dispatch and an
   unknown label never resolves to a default;
6. an SDK snapshot test proving the added surface is exactly the accepted
   contract, with no invented identifier; and
7. the repository-local `just check` with zero issues.

No verification result is claimed on this page. No "Accepted" row here is
evidence that a surface is implemented; acceptance of a contract does not prove
implementation, and this page claims no green Rust or Lua.

## Alternatives considered

| Alternative                                                     | Trade-off                                                                                              | Disposition                                                                                     |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| Treat the candidate Beacon design as if v1 already carried it   | Simpler prose, but implies an API that does not exist and hides every missing surface.                 | Rejected; this page marks each operation Accepted, Provisional, or Blocked with its named task. |
| Invent provisional Beacon and overlay spellings to make the map | Produce a concrete-looking table, but pre-empts `W-01`/`W-29` and creates a competing public contract. | Rejected; no spelling is invented, and provisional Composer names stay marked owner-pending.    |
| Wait for `W-01`/`W-29` before writing any reconciliation        | Avoids blocked rows, but leaves the page set implying an available surface and delays the SDK owner.   | Rejected; the blocking is recorded now so the SDK tasks have a checkable input.                 |
| Fold Beacon needs into `W-139` (history/search/selection)       | Reuses one SDK task, but merges unrelated public contracts and hides the Beacon block.                 | Rejected; `W-139` is named only as an adjacent surface owner.                                   |

## Affected contracts

| Contract                                                                                                                                                   | Effect                                                                      |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted)                                                                        | Consumed as the authority for accepted spellings; not modified.             |
| [Plugin Platform RFC](plugin-platform-rfc.md) (accepted)                                                                                                   | Consumed for the capability families and command registry; not modified.    |
| [Beacon Targeting Framework (Candidate)](beacon-targeting-framework-candidate.md) (draft)                                                                  | Its candidate operations are mapped; its direction is not promoted.         |
| [Beacon plugin page set](../docs/plugins/beacon/design.md) (draft)                                                                                         | Its policy operations are reconciled here; the page set is not modified.    |
| [Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md) (draft) | Consumed for the accepted split and fences; its host API stays with `W-29`. |
| [Composer Architecture and Host API](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md) (accepted)   | Consumed for the provisional overlay/input-capture shape; not modified.     |
| [ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md) (accepted)                | Consumed as the accepted split and task routing; not modified.              |

## Open points

None of these is a new global open question; each is parked with its named
owner.

- **`OQ-056` host-API scope**, including the focusable-overlay and transient
  input-capture host API and every Beacon-facing capability dimension and
  version, stays **Open** with `W-01` (Issue #396). This page does not decide
  it.
- **`W-29` host names, capability identifiers, and version** that expose
  `TargetEngine`, `AnnotationEngine`, and the dispatch bridge stay with `W-29`.
- **`OQ-088` session behavior** (scope defaults per operator, label overflow and
  handedness, session timeout, invalidation, cross-workspace behavior) stays
  Open with its owner.
- **`OQ-052` Leader and chord namespace** stays Open; the default Leader and
  chord vocabulary are illustrative-only in the candidate page.
- **Menu surface ownership**: whether the target-first context menu is a Core
  surface or plugin-composed depends on `W-29`/`W-120`.
- **Badge and theme-token namespace**: the badge tokens and their theme mapping
  stay candidate and depend on `W-29`/`W-120`.

## Acceptance criteria

1. Every Beacon policy operation is mapped to an accepted surface, a
   provisional surface with its owner, or an explicit block naming the blocking
   task.
2. Missing SDK methods, capability declarations, quotas, and event/dispatch
   semantics are stated, each with an owner or blocking task, and no identifier
   is invented.
3. Incompatible assumptions between the Beacon policy contract and the accepted
   v1 and provisional overlay surfaces are stated and parked, not silently
   chosen.
4. The no-hidden-privilege statement is explicit: public capability-gated
   surfaces only, no raw PTY/GPU/window handle, no input hot-path callback, and
   no private first-party bypass.
5. Negative-path behavior is fail-closed with the accepted error vocabulary,
   and no failure falls back to a default target, a truncated snapshot, a
   silent dispatch, or a privileged path.
6. The SDK tasks (`W-120`/`CTX-0065`, `W-43`, `W-139`/`CTX-0066`), the host-API
   tasks (`W-01`, `W-28`, `W-29`), and the Core/plugin tasks (`W-102`, `W-30`,
   `W-121`, `W-53`) are named as owners without deciding their content.
7. `OQ-056` is cross-linked as Open and not decided here; the page is
   self-contained and makes no implementation claim.
8. `just check` passes with zero issues.

## P0 Review Sign-off

Not signed. This document is **draft** and reconciliation-only. Independent
category-owner, docs-curator, and security review are required before it is
promoted beyond draft; no P0 control is changed by this page.

| Role                 | Scope                                                                       | Requirement                                                             |
| -------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `sdk-owner`          | Operation-to-surface mapping, missing surfaces, and blocking-task accuracy  | Approve; confirms the map matches the accepted and owner-pending state. |
| `security-architect` | Capability boundary, capture, dispatch, safe mode, and target-safety fences | Independent security sign-off required before promotion.                |
| `docs-curator`       | Metadata, links, terminology, self-containment, and status honesty          | Approve; confirms schema, discoverability, and candidate marking.       |

## References

- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) -
  accepted v1 function surface, event set, and exclusions.
- [Plugin Platform RFC](plugin-platform-rfc.md) - accepted manifest,
  capability families, grant lifecycle, command registry, and event pipeline.
- [Plugin Manifest and Capability Grammar Authority](manifest-capability-authority.md) -
  canonical manifest spelling and capability grammar.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) - accepted
  per-plugin isolation and resource ceilings.
- [Beacon Targeting Framework (Candidate)](beacon-targeting-framework-candidate.md) -
  plugin-side candidate direction.
- [Beacon plugin design](../docs/plugins/beacon/design.md), [schemas](../docs/plugins/beacon/schemas.md),
  and [evidence](../docs/plugins/beacon/evidence.md) - the reconciled policy
  contract.
- [Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md) -
  terminal-side draft mechanism contract.
- [Composer Architecture and Host API](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/composer-architecture.md) -
  accepted provisional overlay and input-capture host-API shape.
- [ADR 0018 - Beacon Mechanism/Policy Split and Core Targeting-Mechanism Naming](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md) -
  accepted split, names, and task routing.
- [ADR 0015 - Small-Core Extraction Boundaries](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0015-small-core-extraction-boundaries.md) -
  Boundary 3 and the bootstrap fence.
- [Open question OQ-056](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md) -
  open host-API scope, including focusable overlay and transient input capture.
