---
title: Plugin Ecosystem Model
description: Research-derived design input for plugin taxonomy extension points and Panel-as-host implications from records 039 and 040
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: true
sidebar_order: 32
---

# Plugin Ecosystem Model

> Status: **draft**, research-derived design input recording the conclusions of
> two workspace `research` records — `origin/040.md` (plugin system,
> browser/OS and three-layer framing, extension platforms) and `origin/039.md`
> lines 1-1258 (Panel, Activity, native UI) — into the plugin corpus. It is
> **not** an accepted contract, an RFC, or an implementation claim.

Unless a statement cites an accepted document with a relative link, every
conclusion below is a **proposal or observation from those records**. Where the
accepted corpus already covers a point, this page links that document.

## 1. Purpose, status, and attribution

- The source records are read-only provenance in the workspace `research`
  repository; they were not edited for this page and stay unmarked because the
  terminal-direction work uses them too.
- `origin/040.md` is the plugin-system record; `origin/039.md` is the
  Panel/activity/native-UI record.
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
  [Isolation and Resource RFC](isolation-resource-rfc.md) (`IR-D2`).

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
[Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md); a formal
extension-point registration model is not addressed there.

## 5. Manifest expression candidates

### Accepted dependency declaration

The `[dependencies]` table is **accepted** in the
[Plugin Platform RFC accepted manifest schema](plugin-platform-rfc.md). An
entry is the string form `"owner.name" = ">=2.0"` or the inline-table form
`"owner.name" = { version = ">=2.0", prerelease = true }`; the version is
validated by the closed resolver grammar, `prerelease` defaults to `false`, and
the table-form convention follows
[ADR 0009](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0009-plugin-api-v1-lua-surface.md).
The inline-table form is **specified but not yet enforced**: the reference host
dependency list and the SDK validator still accept only the string form and
reject the table form
([Plugin Platform RFC](plugin-platform-rfc.md)).

Registry versus manifest asymmetry: the author-facing manifest is the
declaration source, while the registry is an attestation and index service that
only reads and records the dependency edges and compatibility declarations from
it, and is not authoritative for them (see the registry boundaries in the
[Package Follow-up RFC](package-followup-rfc.md)).

### Candidate contribution shapes (unaccepted)

040 sketches two unaccepted manifest shapes for extension contributions; the
accepted corpus defines only what the
[Plugin Platform RFC](plugin-platform-rfc.md) accepted schema states. Neither
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
[Plugin Platform RFC capability model](plugin-platform-rfc.md) and the
containment rules in the
[Isolation and Resource RFC](isolation-resource-rfc.md): grants stay per plugin
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
[Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md)) and services
(`[services.provided]` in the
[Plugin Platform RFC](plugin-platform-rfc.md)); per-extension-platform API
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
the [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md). The
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
[Plugin Platform RFC](plugin-platform-rfc.md) and the
[Isolation and Resource RFC](isolation-resource-rfc.md). Panel state is split
into distinct axes (lifecycle, focus, visibility, interaction, attention)
rather than one enum. For v1 the record keeps the accepted animation
restrictions: only Core-owned chrome animates, plugin shaders and native
in-process effects stay forbidden, and advanced `VisualState + Transition +
Effect` work waits until the widget runtime and isolation are stable.

## 10. Mapping to the existing corpus

| Theme                                             | Existing document                                                                                                                                                                   | Relationship                                                                                                                    |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Five-role plugin taxonomy                         | [Plugin system](../extensibility/plugin-system.md)                                                                                                                                  | Extends; the taxonomy itself is unaddressed here                                                                                |
| Platform/host versus extension plugin             | [Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md)                                                                                                              | Extends; provider ecology is close but does not name host plugins                                                               |
| Extension points and contribution manifest        | [UI Extensibility Architecture](ui-extensibility-architecture.md); none for `[contributes]`                                                                                         | Extends; the inventory exists, a formal extension-point model is unaddressed                                                    |
| Accepted `[dependencies]` manifest                | [Plugin Platform RFC](plugin-platform-rfc.md)                                                                                                                                       | Aligns; accepted schema already defines the dependency shape                                                                    |
| Capability non-escalation                         | [Plugin Platform RFC](plugin-platform-rfc.md), [Isolation and Resource RFC](isolation-resource-rfc.md)                                                                              | Aligns; the dependency-edge framing is new                                                                                      |
| Extension-platform API versioning                 | [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md), [Plugin Platform RFC](plugin-platform-rfc.md)                                                                    | Extends; host API versioning is candidate                                                                                       |
| Plugin graph                                      | [Plugin system](../extensibility/plugin-system.md)                                                                                                                                  | Extends the dependency and service direction                                                                                    |
| Panel as host / Activity stack                    | [UI Extensibility Architecture](ui-extensibility-architecture.md) (P2), [Plugin Roadmap](../product/plugin-roadmap.md)                                                              | Unaddressed here; the accepted sibling Panel Runtime RFC leaves provider details as its open questions (`RFC-OQ-1`..`RFC-OQ-9`) |
| Native UI, widget layer, and application services | [UI Extensibility Architecture](ui-extensibility-architecture.md), [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md), [Plugin Platform RFC](plugin-platform-rfc.md) | Extends the ownership boundaries, v1 slot UI, and capability families                                                           |

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
