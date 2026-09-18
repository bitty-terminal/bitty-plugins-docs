---
title: Plugin Ecosystem Model
description: Research-derived plugin taxonomy public services and framework layering with explicit owner-pending capture for record 053
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: true
sidebar_order: 32
---

# Plugin Ecosystem Model

> Status: **draft**, research-derived design input recording the conclusions of
> workspace `research` records — `origin/040.md` (plugin system,
> browser/OS and three-layer framing, extension platforms) and `origin/039.md`
> lines 1-1258 (Panel, Activity, native UI) — into the plugin corpus. Sections
> 9.6-9.7 add the plugin-generic capture and owner-pending map for `origin/053.md`
> (public services and framework layering). This is **not** an accepted
> contract, an RFC, or an implementation claim.

Unless a statement cites an accepted document with a relative link, every
conclusion below is a **proposal or observation from those records**. Where the
accepted corpus already covers a point, this page links that document.

## 1. Purpose, status, and attribution

- The source records are read-only provenance in the workspace `research`
  repository; they were not edited for this page and stay unmarked because the
  terminal-direction work uses them too.
- `origin/040.md` is the plugin-system record; `origin/039.md` is the
  Panel/activity/native-UI record.
- A third record, `origin/041.md`, is recorded in the sibling
  [Plugin IPC Boundary](plugin-ipc-boundary.md) page because it covers the
  distinct out-of-process and IPC extension-boundary topic.
- Section 11 lists the decisions still required before this becomes contract.

## 2. Plugin taxonomy

Observation (039): once plugins grow past scripts, the single word "plugin"
becomes ambiguous. The record proposes distinguishing five roles:

| Type           | Recorded example                  |
| -------------- | --------------------------------- |
| Plugin         | git integration                   |
| Service        | LSP / notification / credential   |
| Widget         | clock / status / CPU graph        |
| Application    | bitter / Docker / Mail / Telegram |
| Panel Provider | provides an Application surface   |

The same discussion frames Bitty as an application shell and holds "Bitty Core
provides primitives, not applications."

## 3. Platform plugin versus extension plugin

Proposal (040): do not read every plugin as attached directly to Core. The
record distinguishes a **Platform Plugin** (or Host Plugin) from an **Extension
Plugin**. Bitter, Bitty AI, Statusline, and Docker are each itself a Bitty
plugin and also a host — for example, Bitter hosts `bitter-lsp`,
`bitter-treesitter`, and `bitter-git`; Bitty AI hosts `bitty-ai-openai` and
`bitty-ai-memory`; Statusline hosts `statusline-git`; Docker hosts
`docker-compose`.

The layering is semantic, not a nesting of runtimes:

- **NO nested Lua VMs.** The record explicitly rejects
  `Bitty -> Bitter Lua VM -> bitter-lsp Lua VM` because lifecycle, error
  propagation, permissions, and hot reload would all become harder.
- Every plugin is a **peer** managed by one Bitty plugin runtime; peer
  relations are `dependency`, `service`, `extension point`, and
  `contribution`.
- Summarized as **runtime flat, semantics layered**, consistent with the
  accepted one-VM-per-plugin-identity-and-generation rule in the
  [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (`IR-D2`).

## 4. Extension points as a first-class concept

Proposal (040): beyond today's commands, events, services, UI, and keymaps, the
corpus should introduce **Extension Point** as a first-class concept. A plugin
that hosts a platform declares its own domain points; other plugins contribute
to them. Recorded inventories:

```text
Bitter       bitter.language, bitter.highlighter, bitter.formatter, bitter.completion, bitter.code_action, bitter.status_item, bitter.sidebar
Bitty AI     bitty-ai.model, bitty-ai.tool, bitty-ai.context, bitty-ai.memory, bitty-ai.compactor, bitty-ai.agent, bitty-ai.command, bitty-ai.ui
Statusline   statusline.segment
Docker App   docker.action, docker.renderer, docker.inspector
```

The recorded consequence is that the plugin system stops being "load Lua files"
and becomes "compose different extension graphs". The accepted inventory of
existing extension mechanisms is in the
[UI Extensibility Architecture](ui-extensibility-architecture.md) and the
[Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md); a formal
extension-point registration model is not addressed there.

## 5. Manifest expression candidates

### Accepted dependency declaration

The `[dependencies]` table is **accepted** in the
[Plugin Platform RFC accepted manifest schema](../specifications/plugin-platform-rfc.md). An
entry is the string form `"owner.name" = ">=2.0"` or the inline-table form
`"owner.name" = { version = ">=2.0", prerelease = true }`; the version is
validated by the closed resolver grammar, `prerelease` defaults to `false`, and
the table-form convention follows
[ADR 0009](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0009-plugin-api-v1-lua-surface.md).
The inline-table form is **specified but not yet enforced**: the reference host
dependency list and the SDK validator still accept only the string form and
reject the table form
([Plugin Platform RFC](../specifications/plugin-platform-rfc.md)).

Registry versus manifest asymmetry: the author-facing manifest is the
declaration source, while the registry is an attestation and index service that
only reads and records the dependency edges and compatibility declarations from
it, and is not authoritative for them (see the registry boundaries in the
[Package Follow-up RFC](../packaging/package-followup-rfc.md)).

### Candidate contribution shapes (unaccepted)

040 sketches two unaccepted manifest shapes for extension contributions; the
accepted corpus defines only what the
[Plugin Platform RFC](../specifications/plugin-platform-rfc.md) accepted schema states. Neither
shape below is accepted, and neither matches the accepted owner-qualified
`[plugin] id` grammar:

```toml
[plugin]
name = "bitter-lsp"
version = "0.1.0"

[dependencies]
bitter = ">=0.3"

[contributes]
extensions = [
    "bitter.language-provider",
    "bitter.diagnostics-provider",
    "bitter.completion-provider",
]
```

```toml
[[extensions]]
point = "bitter.language"
id = "lsp"
```

The recorded intent is that the plugin manager can resolve a dependency tree
like an ordinary package manager.

## 6. Dependency must not become capability escalation

Principle (040), stated as a rule worth fixing early: **a dependency
relationship must not become capability escalation.** A host's permissions are
not inherited by the extension. Recorded contrast:

- `bitter-theme-catppuccin` needs only `bitter.theme`.
- An LSP plugin may need `process.spawn` and `filesystem.read`.
- A Git plugin may need `process.spawn: git`.

Without this rule, `evil-plugin -> depends on bitter` would indirectly inherit
every Bitter capability and the sandbox would lose its meaning. This extends the
accepted deny-by-default capability model in the
[Plugin Platform RFC capability model](../specifications/plugin-platform-rfc.md) and the
containment rules in the
[Isolation and Resource RFC](../runtime/isolation-resource-rfc.md): grants stay per plugin
identity and manifest hash, and a dependency edge is not a grant.

## 7. Extension-platform API versioning

Candidate (040): once a plugin can extend a plugin, the stable API surface is
no longer only the Bitty API. The record proposes versioning each host's
extension API explicitly:

```text
bitter.editor@1
bitty-ai.tools@1
bitty-ai.context@2
```

A host can then refactor internals without breaking the ecosystem as long as
the versioned contract is unchanged. The accepted corpus already versions the
Plugin API itself (`compat.plugin-api`, `bitty.api_version` in the
[Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)) and services
(`[services.provided]` in the
[Plugin Platform RFC](../specifications/plugin-platform-rfc.md)); per-extension-platform API
versioning is the additive candidate those records do not yet define.

## 8. The plugin graph

Observation (040): the result is a plugin _tree_ by intent, but from the plugin
manager's point of view it is a **dependency + service + extension graph**, not
a physical parent/child process tree. The record proposes two principles:

> **Every plugin may be an application, and every application may expose its
> own extension platform.**
>
> **Bitty extends plugins; plugins extend ecosystems.**

Bitter, Bitty AI, Statusline, and Docker can then each grow their own ecosystem
without Core expanding with every domain. This extends the current dependency
and service direction in the
[Plugin system](../extensibility/plugin-system.md) contract.

## 9. Panel and activity implications for plugin authors

These are proposals from 039 that build on the accepted Panel Runtime RFC in the
sibling `bitty-terminal-docs` repository (Panel is a generic
workspace-managed application container, not an OS window or a PTY, and
`PanelId != ViewId != TerminalId`). They are not accepted in this repository.

### 9.1 Panel is a host; Terminal is only one Activity

The recorded principle moves from "Panel is not Terminal" to **"Panel is a
host; Terminal is only one Activity."** Terminal, native application, rich,
canvas, and helper content can all live in a panel. Presentation modes
(`tiled`, `floating`, `overlay`, `fullscreen`, `scratchpad`, `pinned`,
`popover`) are runtime properties, not panel types, so one panel identity keeps
its lifecycle, input, and surface across mode transitions.

### 9.2 Activity stack and session survival

An **Activity** layer above presentation mode uses
`panel:push(activity)` / `panel:pop()` semantics: a native application activity
covers a terminal activity without killing it, so popping back restores the live
shell and its history. This works because the accepted runtime keeps
`TerminalRegistry` as the PTY lifecycle owner rather than the panel, so the PTY
never dies when it is hidden.

### 9.3 Document, View, and Panel are distinct

Recorded as `Document != View != Panel`, following the Emacs buffer/window
model: a document is an editable object that may be shown by zero or more views,
and a view owns only display state. A native UI bypasses the terminal cell grid
entirely (`Lua Plugin -> Bitty UI Tree -> Layout -> wgpu`) and is
**Bitty-native, not OS-native**. The record splits editor responsibilities so
"bitter" can be a Lua application while heavy mechanisms stay in Rust:

| Rust core primitive                  | Lua application                           |
| ------------------------------------ | ----------------------------------------- |
| Text buffer, cursor, selection       | Vim-like mode, Normal/Insert, Visual mode |
| IME, grapheme segmentation           | keymap, commands                          |
| Unicode shaping                      | editor behavior                           |
| undo/redo engine                     | workflows                                 |
| viewport, virtualized text rendering | UI composition, plugins                   |
| clipboard primitive                  | user commands                             |

### 9.4 Native Widget Layer progression

v1 stays the accepted declarative slot UI: `bitty.ui.mount` / `bitty.ui.update`
with a closed slot set and only `Text`, `Row`, `Column`, and `List` nodes, per
the [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md). The
record's proposed later stage is a retained/declarative **Widget Tree**
(including `RichText`, `TextInput`, `Editor`, `Button`, `Toggle`, `Slider`,
`VirtualList`, `Tree`, `Table`, `Tabs`, `ScrollView`, `Canvas`, `Image`,
`Terminal`, `Split`, `Stack`, `Overlay`, `Popover`). Retained/declarative is a
hard recorded preference: Lua exposes state, policy, and application logic
while Rust owns the hot path, mechanism, and rendering, and Lua never enters a
per-frame immediate-mode draw loop.

### 9.5 Application services, capability sandbox, and visual state

Mail, Telegram, and Docker need more than UI. The record lists the required
service surface — `process`, `network`, `fs`, `store`, `secrets`, `tasks`,
`notifications`, `clipboard`, `commands`, `events`, `services` — each behind
the same deny-by-default sandbox described by the
[Plugin Platform RFC](../specifications/plugin-platform-rfc.md) and the
[Isolation and Resource RFC](../runtime/isolation-resource-rfc.md). Panel state is split
into distinct axes (lifecycle, focus, visibility, interaction, attention)
rather than one enum. For v1 the record keeps the accepted animation
restrictions: only Core-owned chrome animates, plugin shaders and native
in-process effects stay forbidden, and advanced `VisualState + Transition +
Effect` work waits until the widget runtime and isolation are stable.

### 9.6 Four-layer framework ecosystem (053, candidate)

Status: **research proposal, not accepted or implemented**. Research record
053 lines 682-1327 extends the earlier semantic layering:

| Layer                      | Proposed responsibility                                                                             | Boundary                                                                     |
| -------------------------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Rust Core and host         | Rendering, PTY, input, platform mechanisms, resource scheduling, lifecycle and security enforcement | No raw internal Rust APIs for ordinary plugins                               |
| Small public Lua SDK       | Stable, general semantic wrappers over explicitly admitted host operations                          | Not a second monolithic Core or a promise that every source namespace exists |
| Optional framework plugins | Replaceable UI composition, reusable domain adapters and higher-level authoring abstractions        | Independent versions, ordinary plugin grants, no special runtime privilege   |
| Application plugins        | User workflows, dashboards, tools and application presentation                                      | Consume public contracts rather than peer source trees                       |

This is an **abstraction/stability** model, distinct from the Pure Lua / System
CLI / Plugin Service / Native Helper **reuse** layers in
[Plugin Reuse and Provider Ecology](../packaging/plugin-reuse-and-providers.md). It does not
introduce nested VMs: section 3 and the accepted isolation contract still apply.
A framework implemented as a separate plugin uses the public service boundary;
a packaged private Lua helper uses rooted `require`. A source sketch importing
`wheel.tool` or `bitty.ui` does not authorize cross-package module loading.

The source argues for many competing frameworks rather than a mandatory official
framework, and for their evolution without forcing Core or the public SDK to
change. Stability belongs to reviewed public contracts, not exposed renderer,
windowing, IPC, font-engine, or scheduler internals. Keeping that SDK small is
the proposal; the source's `ipc`, `panel`, `process`, `filesystem`, `async`, and
other namespace inventory is not an addition to
[Plugin API v1](../sdk/plugin-api-v1-lua-surface-rfc.md#not-in-plugin-api-v1).
UI-specific alternatives and their limits are recorded under
[Framework-level Lua UI](ui-extensibility-architecture.md#framework-level-lua-ui-053-candidate).

The phrase "Rust mechanisms, Lua composition and policy" is not a transfer of
security policy. Network authorization, secret resolution, process-spawn
permission, scheduler ceilings, and resource enforcement remain host authority;
frameworks can compose controlled requests, never grant themselves those powers.
Nor does 053's HTTP/TLS sketch move network initiation into the terminal Core:
the shared [decision register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/index.md)
(DIR-016/DIR-017) and
[security invariants](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md#security-invariants)
remain authoritative. Source suggestions that those mechanisms could be
independent plugins do not permit ambient network, plaintext secrets, raw
spawn, or unrestricted scheduling.

### 9.7 Research 053 coverage and owner handoff

This is documentation capture only: no API acceptance, product implementation,
or ownership expansion. The immutable source was read in full (lines 1-1328).
The following map distinguishes plugin-generic capture from domain conclusions
that this corpus cannot capture on behalf of excluded owners.

| Source lines and theme                                                                                         | Destination in this corpus                                                                                                                                         | Disposition                                                                                                                                                                                  |
| -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1-209, 564-680: private modules versus public services, typed contracts, public lookup/registration vocabulary | [Cross-package contracts](../packaging/plugin-reuse-and-providers.md#cross-package-contracts-053-candidate), Layer 3                                               | Captured as generic proposal with accepted-boundary reconciliation; source API spellings are not adopted                                                                                     |
| 211-330: install dependencies versus required services, replaceable providers                                  | Same Layer 3 subsection                                                                                                                                            | Captured as distinction and open declaration/selection contract, not new manifest syntax                                                                                                     |
| 332-448, 564-680: local/remote proxies and async-first calls                                                   | [Local and remote service proxies](plugin-ipc-boundary.md#13-local-and-remote-service-proxies-053-candidate)                                                       | Captured as candidate; bounded synchronous v1 calls and VM marshalling retained                                                                                                              |
| 450-499: uniform streaming over different transports                                                           | Same proxy section                                                                                                                                                 | Generic bounded stream proposal captured; model event semantics pending AI/Wheel owner                                                                                                       |
| 682-921, 1116-1327: four layers, small SDK, replaceable frameworks, stable public boundary                     | Section 9.6 and [Framework-level Lua UI](ui-extensibility-architecture.md#framework-level-lua-ui-053-candidate)                                                    | Captured as proposal, including immediate-mode conflict and independent framework iteration                                                                                                  |
| 72-209, 268-330, 450-499, 975-1020: model APIs, provider substitution and normalized model streams             | This handoff; existing [Model-provider direction](../packaging/plugin-reuse-and-providers.md#model-provider-direction-candidate) is context, not 053 owner capture | Pending AI/Wheel owner capture: model list/resolve/generate/stream semantics; model selection; vendor transport adaptation; start/text/reasoning/tool-call/usage/finish/error event meanings |
| 502-560, 922-973, 1024-1065: user/project tool discovery and framework authoring                               | This handoff                                                                                                                                                       | Pending AI/Wheel owner capture: tool registry, discovery/trust of installed versus user/project packages, schema/permission DSL, host execution wrapper and tool-result contract             |
| 1069-1112, 653-674, 1116-1157: agent/context/workflow framework and dashboard composition                      | This handoff                                                                                                                                                       | Pending AI/Wheel owner capture: spawn/task/context/checkpoint/message/tool abstractions, event semantics, delegation and scheduling authority; no agent API accepted here                    |

The domain rows are distinct conclusions, not expendable examples. Replacing
them all with a generic service proposal would lose their intended model, tool,
and multi-agent contracts. `Wheel` and its package names remain source
vocabulary, not an asserted repository roster or accepted ownership decision.
The commander must request scope expansion or obtain capture from the relevant
AI/Wheel owner, including an explicit disposition of those proposals. No changes
to `bitty`, `bitty-ai`, `bitty-devtools`, Wheel, or their docs are authorized by
this capture; their responsibilities are not reassigned here.

**Archive status: Captured as record provenance; owner capture not asserted.**
The research archive marked records 044 through 055 Captured and renamed the
originals with a `.completed` suffix on 2026-09-18 under an owner directive
that makes the `*-docs` corpora the working corpus and keeps the archive as
discussion provenance. That rename records archive-level status, not
owner-verified capture in this corpus: the AI/Wheel owner rows above remain
pending, and this page asserts no capture claim for them. Future contract
acceptance can remain open after faithful owner capture; missing owner capture
cannot be hidden by calling the whole record a generic proposal.

## 10. Mapping to the existing corpus

| Theme                                             | Existing document                                                                                                                                                                                            | Relationship                                                                                                                    |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| Five-role plugin taxonomy                         | [Plugin system](../extensibility/plugin-system.md)                                                                                                                                                           | Extends; the taxonomy itself is unaddressed here                                                                                |
| Platform/host versus extension plugin             | [Plugin Reuse and Provider Ecology RFC](../packaging/plugin-reuse-and-providers.md)                                                                                                                          | Extends; provider ecology is close but does not name host plugins                                                               |
| Extension points and contribution manifest        | [UI Extensibility Architecture](ui-extensibility-architecture.md); none for `[contributes]`                                                                                                                  | Extends; the inventory exists, a formal extension-point model is unaddressed                                                    |
| Accepted `[dependencies]` manifest                | [Plugin Platform RFC](../specifications/plugin-platform-rfc.md)                                                                                                                                              | Aligns; accepted schema already defines the dependency shape                                                                    |
| Capability non-escalation                         | [Plugin Platform RFC](../specifications/plugin-platform-rfc.md), [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md)                                                                          | Aligns; the dependency-edge framing is new                                                                                      |
| Extension-platform API versioning                 | [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md), [Plugin Platform RFC](../specifications/plugin-platform-rfc.md)                                                                    | Extends; host API versioning is candidate                                                                                       |
| Plugin graph                                      | [Plugin system](../extensibility/plugin-system.md)                                                                                                                                                           | Extends the dependency and service direction                                                                                    |
| Panel as host / Activity stack                    | [UI Extensibility Architecture](ui-extensibility-architecture.md) (P2), [Plugin Roadmap](../product/plugin-roadmap.md)                                                                                       | Unaddressed here; the accepted sibling Panel Runtime RFC leaves provider details as its open questions (`RFC-OQ-1`..`RFC-OQ-9`) |
| Native UI, widget layer, and application services | [UI Extensibility Architecture](ui-extensibility-architecture.md), [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md), [Plugin Platform RFC](../specifications/plugin-platform-rfc.md) | Extends the ownership boundaries, v1 slot UI, and capability families                                                           |

## 11. Research open items

These are **research open items, not accepted open questions**. Each must be
decided in the owning contract before any conclusion here becomes contract:

1. Extension-point registration shape: how a host declares a point, how IDs
   are namespaced, and how conflicting contributions resolve.
2. `[contributes]` / `[[extensions]]` manifest schema: whether contributions
   are declared statically, validated before activation, and how they relate to
   the accepted owner-qualified `[plugin] id` and `[dependencies]` grammar.
3. Extension-platform API versioning: the grammar and compatibility policy for
   identifiers such as `bitter.editor@1` and `bitty-ai.context@2`.
4. Activity layer contract: push/pop semantics, session survival, and how the
   panel provider surface interoperates with the accepted Panel Runtime RFC and
   its open questions (`RFC-OQ-1`..`RFC-OQ-9`).
5. Native Widget Layer timeline: when the retained widget tree follows the v1
   declarative slot UI, and the exact retained/declarative contract.
6. Visual-state model granularity and the conditions for opening advanced panel
   effects beyond the accepted bounded Core animations.
7. Ownership: which repository owns each contract (this corpus versus the
   terminal or AI documentation repositories) once a decision is proposed.
