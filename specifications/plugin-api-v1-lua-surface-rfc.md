---
title: Plugin API v1 Lua Surface RFC
description: Draft contract resolving the Plugin API v1 Lua module functions payloads event names and L1/L2 split under OQ-011
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: true
sidebar_order: 29
---

# Plugin API v1 Lua Surface RFC

## Status

**Draft** for review under `bitty-docs/CTX-0143`. This RFC is a proposal, not an
accepted contract: acceptance is a project decision that this draft does not
make. It does not describe implemented behavior, does not authorize shipped,
stable, or compatibility-guaranteed behavior, and does not weaken any normative
security control.

The draft resolves the deferred "final spelling" left open by the
[Plugin Platform RFC](plugin-platform-rfc.md) for OQ-011 and the three
conflicting candidate spellings recorded in the corpus:

| Candidate                                                        | Recorded in                                                                                                 |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `bitty.commands.*`, `bitty.events.*`, `bitty.ui.*` namespaces    | [Plugin Platform RFC host namespaces](plugin-platform-rfc.md) (explicitly illustrative)                     |
| `bitty.api.register_panel/on_event/get_terminal_state`           | `bitty-docs` finding `FIND-0003` `ECO-SDK-01` (recorded in the shared checkout; not yet on `origin/main`)   |
| `register_panel/on_event/get_terminal_state` (R-SDK-1 gate list) | `bitty` CarryCtx note `PX-1199` (CTX-0221 first-plugin-batch plan, planning only)                           |
| `bitty.services:get(...)` colon-style methods                    | [Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md) (Draft, post-1.0 provider follow-up) |

Evidence revisions inspected read-only for this draft: `bitty` `1ea2f66`
(local checkout; `bitty-plugin-host` and `bitty-lua` sources; the workspace was
behind `origin/main` at inspection time), `bitty-plugin-sdk` worktree CTX-0015
branch `ctx-0015/feat-manifest-lint` at `d2cad1f` (manifest/lint in review, not
accepted). The SDK produces no authoritative surface: per
[core boundaries](../architecture/core-boundaries.md) and the Plugin Platform
RFC, an SDK surface must derive from an accepted host contract.

### Authority tension this RFC does not resolve by itself

Two accepted statements constrain where the surface becomes authoritative:

1. [Core boundaries](../architecture/core-boundaries.md) records that the
   authoritative Plugin API definition lives in the core repository and the SDK
   is generated output.
2. [Lua Runtime RFC](lua-runtime-rfc.md) says the single host bridge in every VM
   is a versioned `bitty` module and that "its function surface is owned by the
   respective API RFCs"; [ADR 0006](../decisions/adrs/ADR-0006-os-env-policy.md)
   already fixes `bitty.env.get` and `bitty.env.has` under that module.

This RFC proposes that the accepted surface text lives in the `bitty-docs`
contract corpus while `bitty` implements it and the SDK is generated from it.
That proposal is open question [LUA-OQ-1](#open-questions). If the project keeps
definition authority in the core repository, this RFC is the reviewed input
contract to be mirrored there without divergent wording.

## Purpose and scope

In scope: the Lua module name, namespace layout, v1 function and field
spellings, function signatures, argument and payload schemas, the closed v1
event-name set with payload shapes, the L1/L2 extension-level split, and an
explicit exclusion list.

Out of scope; owned elsewhere and only referenced here:

- Manifest format, capability identifier grammar, grant lifecycle, and the event
  pipeline classes, batching, budgets, and drop policy
  ([Plugin Platform RFC](plugin-platform-rfc.md), accepted).
- VM construction, restricted standard library, rooted module resolution,
  diagnostics classes ([Lua Runtime RFC](lua-runtime-rfc.md), accepted), pins and
  allowlist ([ADR 0005](../decisions/adrs/ADR-0005-lua-pins-and-stdlib.md)),
  environment reads ([ADR 0006](../decisions/adrs/ADR-0006-os-env-policy.md)),
  async boundary and tasks/timers ([ADR 0007](../decisions/adrs/ADR-0007-async-gc.md)).
- Resource ceilings and enforcement numbers
  ([Isolation Resource RFC](isolation-resource-rfc.md), accepted).
- Scene content contract ([Rich Presentation RFC](rich-presentation-rfc.md),
  accepted).
- Panel identity, lifecycle, and providers
  ([Panel Runtime pre-study](panel-runtime-pre-study.md), Draft) and workspace
  layout ([Workspace Compositor](workspace-compositor.md), accepted without a
  `PanelId`).

## Normative sources this proposal must not weaken

- [Security overview](../security/overview.md): untrusted-by-default posture,
  capability families, invariants 2 (no ambient authority), 3 (presentation,
  never Terminal Truth), 4 (no hot-path execution), 8 (updates cannot silently
  add capabilities), and 10 (`bitty --safe`).
- [Threat model](../security/threat-model.md): T-06, T-07, T-10, T-12, T-13 and
  the plugin-to-host data-flow controls.
- [Core boundaries](../architecture/core-boundaries.md): mechanism/policy split,
  observation-versus-interception, declarative UI, generation ownership, and the
  two security domains.
- [Plugin system](../extensibility/plugin-system.md): extension levels 1-4,
  register-versus-claim, qualified naming, key-binding precedence, and the
  governing boundary that plugins alter presentation but never Terminal Truth.
- [Plugin Platform RFC](plugin-platform-rfc.md): accepted manifest, capability
  identifiers, v1 surface coverage, namespace rules, and event pipeline.

This RFC selects spellings for controls the sources already accept. It moves no
requirement between owners and relaxes no gate.

## Candidate resolution

The surface adopts the accepted `bitty` module root and one spelling per
concept. Options were compared against the accepted sources and the Rust
`bitty-plugin-host` evidence (`crates/bitty-plugin-host/src/{event,registry,host,capability,manifest}.rs`),
which is the only exact, machine-checkable representation today.

| Option                                                        | Disposition     | Rationale                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Module root `bitty`, namespaced functions                     | **Adopted**     | Accepted by [Lua Runtime RFC](lua-runtime-rfc.md) ("the single host bridge in every VM is a versioned `bitty` module") and already used by accepted `bitty.env.get`/`bitty.env.has` in [ADR 0006](../decisions/adrs/ADR-0006-os-env-policy.md). The Plugin Platform RFC namespace rules give each namespace an accepted contract.                                                            |
| Module root `bitty.api.*`                                     | Rejected        | Adds an unaccepted nesting level with no contract behind it; conflicts with the accepted `bitty.env.*` shape; would force one concept to have two spellings. Only source is a finding recommendation that itself cites no accepted spelling.                                                                                                                                                 |
| Flat `bitty.register_command`/`on_event`/`get_terminal_state` | Rejected        | Accepted material uses namespaced shapes (`bitty.commands.register`, `bitty.events.subscribe`, `bitty.terminal.snapshot`); flat verbs consume the global module namespace, collide with future accepted additions (`bitty.env`), and lose the per-namespace capability mapping.                                                                                                              |
| `register_panel` for a panel provider                         | Rejected for v1 | Panel identity and lifecycle are not accepted: [Workspace Compositor](workspace-compositor.md) explicitly introduces no `PanelId`, and the [Panel pre-study](panel-runtime-pre-study.md) leaves the provider contract and `panel.*` mapping open. Panel providers are post-v1.0 per the [plugin roadmap](../product/plugin-roadmap.md#post-v10-panel-ecosystem-candidates) pending that RFC. |
| Colon-style methods `bitty.services:get(...)`                 | Rejected for v1 | Accepted material uses dot calls with explicit option tables; colon methods imply Lua object/self semantics that the host-owned value-return contract does not require. Provider ecology remains Draft post-1.0.                                                                                                                                                                             |

`register_panel`/`on_event`/`get_terminal_state` reappear in this surface as
`bitty.ui.mount`, `bitty.events.subscribe`, and `bitty.terminal.snapshot`
respectively; the L2 coverage those names targeted is preserved without
pre-empting the panel contract.

## Module, loading, and versioning

1. `bitty` is a host-owned table injected into every VM (configuration, system,
   and per-plugin) at construction; it is not loaded through `require`, because
   rooted module resolution never reaches host internals.
2. The table and its sub-tables are read-only from Lua. Assignment or raw
   metatable mutation fails with a typed `runtime` diagnostic. No plugin may
   replace, wrap, or shadow `bitty`.
3. `bitty.api_version` (proposed) is a SemVer 2 string identifying the host
   bridge line, initially `1.0.0`; minor versions are additive only and removing
   or narrowing a function requires a major version, matching the accepted
   `compat.plugin-api = "^1.0"` policy in the Plugin Platform RFC.
4. There is no `bitty.api` alias, no global function outside `bitty`, and no
   second spelling for any v1 concept.
5. Namespaces that are not granted are either absent from the VM or present and
   fail closed with a typed denial. The proposal is fail-closed typed denial for
   consistency with [ADR 0006](../decisions/adrs/ADR-0006-os-env-policy.md);
   exact absence-versus-stub behavior is [LUA-OQ-2](#open-questions).

## Extension-level split

The accepted [extension levels](../extensibility/plugin-system.md) name Level 1
Control, Level 2 UI, Level 3 Presentation, and Level 4 Protocol. Plugin API v1
covers Level 1 fully, a minimal Level 2, and read-only terminal semantics; it
excludes Levels 3 and 4.

| Level               | v1 status | Surface element                                                       | Capability gate                              |
| ------------------- | --------- | --------------------------------------------------------------------- | -------------------------------------------- |
| L1 Control          | Included  | `bitty.commands.register`                                             | none (commands are core-registered behavior) |
| L1 Control          | Included  | `bitty.events.subscribe` (lifecycle and observation)                  | none; payload access stays bounded           |
| L1 Control          | Included  | `bitty.keymaps.suggest`                                               | none (suggestion only)                       |
| L1 Control          | Included  | `bitty.settings.get` / `bitty.settings.set`                           | none; plugin-owned namespace                 |
| L1 Control          | Included  | `bitty.store.get` / `bitty.store.set`                                 | none; quota-bounded                          |
| L1 Control          | Included  | `bitty.notify.show`                                                   | `platform.notify`                            |
| L1 Control          | Included  | `bitty.env.get` / `bitty.env.has` (already accepted in ADR 0006)      | `env:<KEY>` for plugins                      |
| Cross-cutting       | Included  | `bitty.services.get` (consumer side)                                  | none; provider grants stay with the callee   |
| L2 UI               | Included  | `bitty.ui.mount` / `bitty.ui.update` (declarative slot contributions) | `ui.rich`; `ui.overlay` for the overlay slot |
| L2 UI / observation | Included  | `bitty.terminal.snapshot` (`scope = "semantic"` only)                 | `terminal.semantic-read`                     |
| L3 Presentation     | Excluded  | decorations, annotations, highlighting, replacement                   | —                                            |
| L4 Protocol         | Excluded  | OSC/APC and structured-output handler registration                    | —                                            |

Interception handlers are part of L1 event subscription but deliver only the
bounded metadata below; they are not a separate level.

## Function surface (proposed spellings)

Signatures use Lua notation. `?` marks optional fields; every table is a plain,
bounded data table, never a host object handle. All registration calls are valid
only while the plugin generation is activating; after activation returns,
further registration is a registration error. Spawned resources are owned by
`(PluginId, generation)` and disposed with it.

### Commands

```lua
bitty.commands.register(def) -> handle
```

| `def` field   | Type     | Required | Proposed rule                                                                                                                    |
| ------------- | -------- | -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `id`          | string   | yes      | Plugin-local command segment, `^[a-z][a-z0-9-]{0,63}$`; host qualifies to `<plugin-id>:<id>`                                     |
| `title`       | string   | yes      | Bounded display text, host-rendered, never markup                                                                                |
| `description` | string   | no       | Bounded display text                                                                                                             |
| `params`      | table    | no       | Map of argument name to `{ type, required?, values?, description? }`; `type` in `string \| integer \| number \| boolean \| enum` |
| `result`      | table    | no       | Declared result shape for CLI, palette, IPC, and Agent reuse                                                                     |
| `run`         | function | yes      | `function(args) -> result`; `args` is a validated plain table                                                                    |

The qualified name must already be reserved through the manifest
(`[lazy].commands`), and duplicate qualified names across plugins are rejected
at graph construction, not shadowed. The `params`/`result` metadata representation
is [LUA-OQ-3](#open-questions): the accepted RFC requires it, but neither the
accepted manifest schema nor `bitty-plugin-host` models it yet.

### Events

```lua
bitty.events.subscribe(name, handler) -> handle
```

- `name` must be one of the closed v1 names in
  [Event names and payloads](#event-names-and-payloads) and must be declared for
  the plugin; subscribing to an undeclared type is a registration error.
- `handler` is `function(event)` where
  `event = { kind = string, sequence = integer, payload = table }`; payloads are
  immutable copies, not live core objects.
- Observation and lifecycle handlers: the return value is ignored.
- Interception handlers: return `false` to veto, anything else to approve.
  Rewriting content is not expressible.
- Coalescing, queue bounds, drop policy, batching, and failure policy are the
  accepted [Plugin Platform RFC pipeline](plugin-platform-rfc.md#event-pipeline-oq-013)
  and are not restated here.

### Key-binding suggestions

```lua
bitty.keymaps.suggest(def) -> handle
```

`def = { chord = string, command = string, when? = string }` where `command`
names a registered plugin command. Suggestions never override user or workspace
mappings; the accepted precedence
(`user > workspace > first-party/default > plugin suggestion`) applies. The
namespace name and chord grammar have no accepted spelling yet, so this is
[LUA-OQ-5](#open-questions).

### Settings and storage

```lua
bitty.settings.get(key) -> value
bitty.settings.set(key, value) -> boolean
bitty.store.get(key) -> value | nil
bitty.store.set(key, value) -> boolean
```

- Settings keys are dot paths relative to `plugins.<owner>.<name>`; plugins
  cannot read or write outside their own namespace. Typed schema declaration,
  merge, and reload semantics are owned by the
  [Configuration Model RFC](configuration-model-rfc.md) (OQ-010).
- Storage is the quota-bounded key-value area scoped by plugin ID and
  generation and persisted under the platform data directory. Values are bounded
  plain data. Quota numbers belong to the [Isolation Resource RFC](isolation-resource-rfc.md);
  value encoding and the generation/reload interaction are
  [LUA-OQ-6](#open-questions).
- Neither namespace grants filesystem authority; `fs.*` grants remain a
  separate capability path that v1 does not define a Lua entry point for.

### Notifications and environment

```lua
bitty.notify.show(payload) -> boolean
bitty.env.get(name) -> string | nil
bitty.env.has(name) -> boolean
```

`payload = { title = string, body? = string, urgency? = "low"|"normal"|"critical" }`,
gated by `platform.notify` and subject to host rate policy. The `bitty.env.*`
contract is accepted in [ADR 0006](../decisions/adrs/ADR-0006-os-env-policy.md)
and is referenced, not redefined.

### UI contributions (L2)

```lua
bitty.ui.mount(slot, component) -> handle
bitty.ui.update(handle, component) -> boolean
```

- `slot` is the accepted closed set:
  `terminal | top | bottom | left | right | tabline | statusline | overlay`.
- `component` is a declarative node table shaped by the accepted
  [`SceneNode` contract](rich-presentation-rfc.md), restricted for v1 to text,
  styled spans, rows, columns, lists, popups, and status components. Image,
  code-block, table, and rule nodes exist in the wider scene contract but are
  not part of Plugin API v1.
- Rich content requires `ui.rich`; the `overlay` slot requires `ui.overlay`.
  There are no global coordinates, shaders, pipelines, glyph injection, native
  windows, or renderer handles.
- `tabline` is an exclusive claim; status components compose. Host layout owns
  placement and decoration. `bitty.ui.update` is a proposed minimal update path
  (the scene contract diffs subtrees); whether v1 mounts immutably and remounts
  instead is [LUA-OQ-7](#open-questions).

### Terminal snapshot (L2, read-only)

```lua
bitty.terminal.snapshot(opts) -> Snapshot
```

`opts = { scope = "semantic" }` is the only v1 scope and requires
`terminal.semantic-read`. `scope = "raw"` is rejected in v1; it would require
`terminal.raw-read` and is explicitly high-risk.

Proposed top-level shape, aligned with the versioned
[terminal snapshot contract](terminal-state-rfc.md) and the
accepted semantic projection (visible text with attributes, cursor, modes,
semantic zones):

| Field        | Type    | Proposed meaning                                                     |
| ------------ | ------- | -------------------------------------------------------------------- |
| `version`    | integer | Snapshot contract version                                            |
| `generation` | integer | Committed-state generation the snapshot reflects                     |
| `width`      | integer | Grid columns in the snapshot region                                  |
| `height`     | integer | Grid rows in the snapshot region                                     |
| `rows`       | array   | `{ text = string, spans = { { start, end, attrs } } }` semantic rows |
| `cursor`     | table   | `{ row, col, visible }`                                              |
| `modes`      | table   | Mode flags, for example `{ alternate_screen = boolean }`             |
| `title`      | string  | Bounded title text                                                   |
| `zones`      | array?  | Semantic-zone metadata derived from OSC 7/133 state                  |

Exact attribute encoding, region selection, and alternate-screen behavior are
[LUA-OQ-4](#open-questions). Snapshots served to automation surfaces carry the
untrusted-observation-data label; there is no write path to grid, cursor, modes,
or scrollback in v1.

### Services

```lua
bitty.services.get(iface, opts) -> service | nil
```

`opts = { version = ">=2.0" }` uses the accepted version-requirement grammar.
The provider is selected before activation or resolution fails; the callee
executes with its own grants, arguments are validated against the interface
schema, and results are values rather than cross-VM handles. Provider-side
registration spelling and interface-schema ownership are
[LUA-OQ-8](#open-questions); v1 consumers exist only once that is defined.

### Tasks and timers

Host-owned tasks and timers are accepted with RC-4 caps (64 tasks / 32 timers
per plugin) in [ADR 0007](../decisions/adrs/ADR-0007-async-gc.md), which writes
`task.spawn` and `timer.create` without a module prefix. This RFC does not fix
that spelling; it is [LUA-OQ-9](#open-questions) and stays outside the frozen v1
surface until reconciled.

## Event names and payloads

The closed v1 name set is exactly the `EventKind` closed set implemented in
`bitty-plugin-host/src/event.rs`; `EventKind::parse` and `as_str` round-trip
these strings. The envelope is
`{ kind = string, sequence = integer, payload = table }`.

| Kind                         | Class        | Proposed Lua payload          | Notes                                                 |
| ---------------------------- | ------------ | ----------------------------- | ----------------------------------------------------- |
| `plugin.activated`           | Lifecycle    | `{}`                          | Delivered to the owning plugin only                   |
| `plugin.suspended`           | Lifecycle    | `{}`                          | Owning plugin only                                    |
| `plugin.disposed`            | Lifecycle    | `{}`                          | Owning plugin only                                    |
| `handler.violation`          | Lifecycle    | `{}`                          | Owning plugin only; diagnostic detail stays host-side |
| `terminal.opened`            | Observation  | `{}`                          | No identity field today; see LUA-OQ-10                |
| `terminal.closed`            | Observation  | `{}`                          | No identity field today; see LUA-OQ-10                |
| `terminal.title-changed`     | Observation  | `{ title = string }`          | Bounded (`EVENT_MAX_BYTES`)                           |
| `terminal.cwd-changed`       | Observation  | `{ cwd = string }`            | Bounded; treat as sensitive-capable display data      |
| `terminal.bell`              | Observation  | `{}`                          | Coalescable events collapse to the latest value       |
| `focus.changed`              | Observation  | `{}`                          | Coalescable                                           |
| `selection.changed`          | Observation  | `{}`                          | Coalescable; no selection text in v1                  |
| `process.exited`             | Observation  | `{}`                          | Exit status is not carried by the host payload today  |
| `config.reloaded`            | Observation  | `{}`                          | —                                                     |
| `intercept.command-dispatch` | Interception | `{ action, origin, preview }` | Bounded sanitized metadata; veto/approve only         |
| `intercept.terminal-spawn`   | Interception | `{ action, origin, preview }` | Bounded sanitized metadata                            |
| `intercept.paste`            | Interception | `{ action, origin, preview }` | Never carries paste text without `clipboard.read`     |
| `intercept.open-url`         | Interception | `{ action, origin, preview }` | Bounded sanitized metadata                            |

Observation handlers receive a bounded copy; they never receive live core
objects. `bitty-plugin-host::HostObservation` also has host-side
`ModeChanged` and `Damage` side-queue variants with no `EventKind` counterpart;
they stay host-internal and are not expressible in the v1 Lua vocabulary, which
is consistent with the accepted no-hot-path-events rule.

## Not in Plugin API v1

1. **Terminal Truth writes.** No grid, cursor, mode, scrollback, or reply
   mutation; no raw output transform.
2. **Raw and input authority.** No `terminal.raw-read`, `terminal.input.self`,
   `terminal.input.all`, `terminal.manage`, or input injection surface.
3. **Hot-path events.** No byte-received, cell-changed, damage, glyph-rendered,
   or per-frame event names.
4. **Level 3 presentation.** No decorations, annotations, semantic
   highlighting, or presentation replacement.
5. **Level 4 protocol registration.** No OSC/APC or structured-output handler
   registration, and no use of `ui.protocol-register`.
6. **Panel providers.** No `register_panel`, `PanelId`, `PanelProvider`, or
   panel lifecycle; panel providers wait for the Panel RFC. `bitty.ui.mount`
   contributes declarative slot content only.
7. **Browser, Agent, MCP, and AI surfaces.** Capability families exist in the
   host crate evidence, but v1 defines no Lua entry points for them.
8. **Interception rewriting.** Veto or approve only.
9. **Ambient privileged services.** No filesystem, process, network, clipboard,
   runtime, or debug call surface in this RFC; capability identifiers exist,
   but their Lua entry points are separate host-service contracts.
10. **Aliases.** No `bitty.api` alias, no flat-verb aliases, and no colon-method
    variants.
11. **Rust internals.** No `bitty.ipc.*`, `bitty.renderer.*`, grid-object
    access, or cross-plugin `require`.

## Compatibility policy

- The module root `bitty`, the namespace names, and the v1 event-name set are
  stable within `1.x`; additions are minor versions.
- Removing or narrowing a function, changing an argument schema incompatibly,
  or removing an event name requires a major version and migration notes.
- The manifest `compat.plugin-api` range is the compatibility gate; the runtime
  `bitty.api_version` and the manifest range must agree at activation.
- Unknown future fields in payload tables are ignored, not errors; unknown
  event names are registration errors, not implicit subscriptions.

## Security alignment and traceability

| Proposed element                                              | Gate it preserves                                                   | Threat/risk IDs          |
| ------------------------------------------------------------- | ------------------------------------------------------------------- | ------------------------ |
| Read-only `bitty` table; typed denials                        | No ambient authority; capability checks cannot be bypassed from Lua | T-06, R-006              |
| Capability-gated `ui.*`, `terminal.snapshot`, `notify`, env   | Deny-by-default capability families; presentation-not-truth         | T-06, T-13, R-006, R-008 |
| No `raw` snapshot scope; no output transform                  | Terminal Truth core-owned; raw read stays high-risk                 | T-13, R-008              |
| Closed event set; no hot-path names                           | No plugin code on parser/render/input hot paths                     | T-07, R-007              |
| Bounded immutable payloads; bounded previews                  | Untrusted-input treatment; no clipboard text without consent        | T-04, T-10, R-004, R-013 |
| Generation-owned handles; registration confined to activation | Fail-closed reload/disposal; no cross-generation state bridge       | R-007, R-009             |
| Declarative UI primitives only                                | Renderer stays replaceable; no GPU/native handles in Lua            | T-13, R-021              |

## Verification plan

Acceptance of this draft requires, at minimum:

1. **Host parity:** every v1 function maps to an existing or explicitly
   scheduled `bitty-plugin-host` operation, and every event name round-trips
   through `EventKind::parse`/`as_str` in the implementation repository.
2. **Manifest agreement:** undeclared event subscriptions and unreserved
   command names are rejected; duplicate qualified names are rejected at graph
   construction.
3. **Negative capability tests:** absent grants produce typed denials, raw
   snapshot scope is rejected, and panel/provider registration is absent.
4. **SDK derivation:** R-SDK-1 `bitty.d.lua` and R-SDK-2 lint are generated from
   this surface after acceptance and may not invent identifiers; template
   R-TPL-1 uses only L1/L2 elements.
5. **Independent review:** category owner, docs curator, and a security reviewer
   accept the surface, the exclusions, and every high-risk boundary.
6. **Documentation synchronization:** on acceptance, the
   [Plugin Platform RFC](plugin-platform-rfc.md) host-namespace section,
   [core boundaries](../architecture/core-boundaries.md) authority statement,
   the [specifications index](README.md), and the CarryCtx task record are
   updated in the same change; no divergent copy is created.

## Open questions

These are unresolved because accepted material does not decide them. They do not
block reviewing this draft; they block acceptance of the affected element.

1. **LUA-OQ-1, authority placement.** Does the accepted surface text live in
   `bitty-docs` (this RFC's proposal) or remain owned by `bitty` per core
   boundaries, with this RFC mirrored as implementation input? A project
   decision is required.
2. **LUA-OQ-2, absent versus denied namespaces.** Should ungranted namespaces be
   absent from the VM table or present with typed fail-closed denials, and does
   the choice interact with the diagnostics contract?
3. **LUA-OQ-3, command metadata.** The accepted RFC requires parameter and result
   schemas for CLI/IPC/Agent reuse, but no accepted manifest or host structure
   models them; where do they live and what is the exact schema?
4. **LUA-OQ-4, snapshot schema.** Exact attribute encoding, region selection
   (visible versus full grid), zone metadata shape, and alternate-screen rules.
5. **LUA-OQ-5, key-binding suggestions.** Namespace spelling (`bitty.keymaps`),
   chord grammar, `when`-context grammar, and whether suggestions are Lua calls
   or manifest declarations.
6. **LUA-OQ-6, storage semantics.** Value type and size bounds, key grammar,
   quota numbers, and how persisted store state interacts with reload and
   generation disposal.
7. **LUA-OQ-7, UI update model.** Is `bitty.ui.update` needed in v1, or does the
   host treat mounting as declarative and remount on change?
8. **LUA-OQ-8, service provider side.** Provider registration spelling, interface
   schema ownership, missing-provider error taxonomy, and the relationship to
   the Draft provider-ecology RFC.
9. **LUA-OQ-9, tasks and timers.** Reconcile the ADR 0007 `task.spawn` /
   `timer.create` names with the module root (for example `bitty.task.spawn` or
   `bitty.timers.create`) and define handle and cancellation semantics.
10. **LUA-OQ-10, observation identity.** The host payload carries no terminal or
    view identity for `terminal.opened`/`closed`, `focus.changed`,
    `selection.changed`, or `process.exited`; decide whether v1 adds bounded
    identity fields and an exit-status field.
11. **LUA-OQ-11, panel and overlay boundary.** Confirm that `bitty.ui.mount` with
    the `overlay` slot stays valid if the Panel RFC redefines overlays, or gate
    it until then.
12. **LUA-OQ-12, plugin entry point.** Accepted material does not define the
    activation entry point that performs registration; candidate `init.lua` from
    the template plan needs its own contract.

## References

- [Plugin Platform RFC](plugin-platform-rfc.md) — accepted manifest,
  capabilities, namespace rules, event pipeline.
- [Lua Runtime RFC](lua-runtime-rfc.md) — accepted `bitty` host bridge, sandbox,
  module resolution, diagnostics.
- [Core boundaries](../architecture/core-boundaries.md) — ownership and
  authority statement.
- [Plugin system](../extensibility/plugin-system.md) — extension levels,
  register-versus-claim, key-binding precedence.
- [Rich Presentation RFC](rich-presentation-rfc.md) — accepted `SceneNode` and
  `RichBlock` contracts.
- [Isolation Resource RFC](isolation-resource-rfc.md) — RC budgets and queue
  ceilings.
- [Workspace Compositor](workspace-compositor.md) and
  [Panel Runtime pre-study](panel-runtime-pre-study.md) — panel identity and
  provider deferral.
- [ADR 0005](../decisions/adrs/ADR-0005-lua-pins-and-stdlib.md),
  [ADR 0006](../decisions/adrs/ADR-0006-os-env-policy.md),
  [ADR 0007](../decisions/adrs/ADR-0007-async-gc.md) — accepted runtime,
  environment, and async contracts.
- `FIND-0003` `ECO-SDK-01` (bitty-docs finding recorded in the shared
  checkout; not yet committed to `origin/main` at draft time) — candidate
  spelling and SDK readiness gap.
- `bitty` `1ea2f66` — `crates/bitty-plugin-host/src/event.rs` (closed
  `EventKind`/`EventPayload`), `registry.rs`, `host.rs`, `capability.rs`,
  `manifest.rs`; `crates/bitty-lua/src/lib.rs` (VM budgets, no host bridge).
- `bitty-plugin-sdk` CTX-0015 `d2cad1f` — manifest/lint work in review, evidence
  only.
