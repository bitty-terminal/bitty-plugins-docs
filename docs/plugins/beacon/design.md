---
title: Beacon plugin design
description: Lua policy contract for the Beacon targeting-policy plugin above the Core TargetEngine and AnnotationEngine mechanism
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 55
---

# Beacon plugin design

> Status: **candidate direction**. No `bitty-terminal/beacon` repository,
> package, or VM exists. The mechanism/policy split, the Core mechanism names
> `TargetEngine` and `AnnotationEngine`, and the extraction of policy to this
> plugin are accepted by bitty-docs
> [ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md)
> (2026-10-03). Everything below is plugin-side **policy direction** over the
> accepted terminal-side mechanism. Exact Lua and host API spellings are
> deferred to `W-29` (Beacon host API) and `W-120` (SDK surface); this page
> fixes none of them and describes no implemented behavior.

## Problem

Operators need workspace-wide keyboard addressing of panels, workspaces,
command blocks, controls, links, and plugin domain entities for jump, focus,
fold, move, and context actions. Embedding a fixed key vocabulary, menu layout,
scope set, badge theme, or provider composition inside Core couples replaceable
presentation to an always-available mechanism. Core should keep only the
mechanism; the vocabulary and composition should be an optional, replaceable,
capability-gated plugin policy.

## Scope

In scope (all candidate):

- the key language that maps chords and operators to the Core targeting
  mechanism;
- the which-key and menu grammar that guides label selection;
- target scopes and filters that bound a session's candidate set;
- theme badges attached to target kinds and scopes;
- provider composition: which registered providers and target kinds an operator
  or scope considers;
- the target-first menu policy built from a target's declared actions.

Out of scope and owned by the terminal-side mechanism or other contracts:

- the Core `TargetEngine` and `AnnotationEngine` internals, target registry,
  semantic snapshots, generation and stale-handle validation, label allocation,
  annotation rendering, transient input capture, and the command-dispatch
  bridge;
- the exact public host names, capability dimensions, and API version (`W-29`,
  related to OQ-056);
- the accepted manifest, grant, lifecycle, command-registry, and event rules
  the plugin obeys as an ordinary package;
- terminal truth, the input hot path, the parser, and the renderer.

## Mechanism and policy split

Core keeps the mechanism and always-available capability; Beacon owns optional
policy. The names below are accepted by ADR 0018; the policy column is
candidate direction.

| Side                                              | Owns                                                                                                                |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Core mechanism `TargetEngine` (accepted name)     | Target registry, semantic target snapshots, provider registration and composition, and the command-dispatch bridge. |
| Core mechanism `AnnotationEngine` (accepted name) | Annotation layer and the `LabelAllocator`.                                                                          |
| Core host mechanisms (not Beacon-named)           | Transient input capture and the command-dispatch bridge, reached only through the public host API.                  |
| Plugin policy `bitty-terminal.beacon` (candidate) | Key language, which-key integration, scopes, filters, theme badges, provider composition, and target-first menus.   |

The terminal-side
[Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md)
owns the mechanism definition. This page owns only the policy that sits above
it and must not restate or weaken the mechanism's fences.

## Key language

The key language is the policy vocabulary that binds chords to operators and
the target-first entry, and then binds the selected label to a typed command.

- Chords are declared as suggestions through the accepted v1
  `bitty.keymaps.suggest` surface, never as forced input mutations. The accepted
  precedence is user mapping over workspace mapping over first-party/default
  mapping over the plugin suggestion.
- An operator family is a named policy group that narrows the candidate target
  set before label allocation. The candidate direction records action-first
  focus, fold, and move families and one target-first entry; the concrete
  family names, chords, and membership are policy configuration, not fixed
  here.
- A key that is not a valid label, prefix, or cancel key fails open to normal
  input behavior; `Esc` and the idle timeout cancel the session. Capture is
  transient, bounded, revocable, and owned by Core, never by the plugin.
- The operator vocabulary, default chords, and the Leader default are open
  governance directions (OQ-052, OQ-088) and are illustrative-only here.

## Which-key and menu grammar

- Which-key is a continuation hint surface that shows the keys or labels valid
  from the current input prefix. Beacon policy decides which hints it
  contributes and in what order; the surface and its rendering are host-owned.
- A label can open a target-first context menu built from the target's declared
  actions. Policy decides menu ordering, grouping, and labels; the menu
  selection resolves to a typed command id.
- Beacon declares menu and hint content as bounded plain data. It does not draw
  labels, annotations, or overlays, and it does not own the annotation layer.
- The exact hint and menu payload shapes depend on `W-29`/`W-120` and are not
  fixed here.

## Scopes and filters

A session constrains its candidate set by scopes so labels stay meaningful and
do not explode across the workspace.

| Scope (candidate)     | Bounds targets to                                                | Default per operator |
| --------------------- | ---------------------------------------------------------------- | -------------------- |
| `CurrentControlGroup` | The focused control group, for example a form or toolbar region. | Policy direction     |
| `CurrentPanel`        | The focused panel.                                               | Policy direction     |
| `CurrentWorkspace`    | The active workspace.                                            | Policy direction     |
| `Window`              | Everything visible in the active OS window.                      | Policy direction     |
| `WorkspaceRail`       | The workspace indicator surface and its entries.                 | Policy direction     |
| `SemanticDomain`      | A named domain, for example git or container objects.            | Policy direction     |

Rules recorded as candidate direction:

- Every operator has a default scope, and every session result is bounded by at
  least one scope. The narrowest meaningful scope first, widening only on
  explicit request, is the recorded direction; the exact default per operator
  stays open (OQ-088).
- Scopes compose: a `SemanticDomain` session can be narrowed by
  `CurrentWorkspace` without defining a new scope kind.
- Scope resolution is a filter over one collected snapshot, not a second
  collection pass, so a session never widens its snapshot implicitly.
- Filters are predicates over public target metadata only, such as target kind
  or semantic domain. The plugin never inspects private plugin state such as
  prompts, secrets, credentials, memory, or internal tables.
- Scope kinds and their spellings are candidate; the scope-kind vocabulary is
  not an accepted contract.

## Theme badges

- A badge is a short, bounded token attached to a target kind or scope and
  mapped by policy to a theme token. Examples of direction: a workspace pill
  badge, an agent-task badge, a command-block badge.
- Beacon supplies badge metadata as public data. The Core annotation layer and
  the host theme own rasterization and appearance; the plugin does not draw
  into the annotation layer and adds no per-label overlay, panel, or native
  window.
- Badge tokens, their theme mapping, and the theme-token namespace are candidate
  and depend on `W-29`/`W-120`; none is fixed here.

## Provider composition

- The mechanism owns provider registration and tier composition. The
  `TargetEngine` ownership of provider registration and composition is
  ADR-accepted; capability-gated registration, cold-path staged collection, and
  the Core, then Plugin, then Derived tier priority remain mechanism-contract
  direction (draft, `W-29`), not accepted detail.
- Beacon policy composes **selection**: which registered providers and target
  kinds an operator or scope considers, and in what order their targets appear
  before filtering. Selection grants no authority; registering or offering a
  target adds addressability only.
- A provider fault is isolated by the mechanism; Beacon policy cannot widen the
  snapshot or resurrect a failed provider's targets.
- The provider-registration capability identifier, its dimensions, and its API
  version remain open (OQ-056, `W-29`).

## Target-first menu policy

- The target-first grammar labels every in-scope target, then opens a context
  menu built from the selected target's declared action metadata.
- Policy owns the menu shape: which declared actions appear, their grouping,
  ordering, and display labels. A target's declared actions are metadata, not
  authority.
- Selecting a menu entry completes `Action(TargetRef)` by resolving a typed
  command id that executes through the accepted command registry under the
  target owner's capability grants. Beacon executes nothing and forwards no
  authority.
- Whether the context menu is a Core surface or plugin-composed remains open;
  the menu payload shape depends on `W-29`/`W-120`.

## Public API use and forbidden paths

This policy uses only the public, capability-gated host API:

- the `W-29` Beacon host API for `TargetEngine`, `AnnotationEngine`, and the
  command-dispatch bridge, and the `W-01` focusable-overlay and
  transient-input-capture host API for a session;
- the accepted Plugin API v1 surface for commands, events, key suggestions,
  settings, notifications, declarative UI slots, and services, with a new
  surface added at `W-120` only after its contracts are accepted;
- the accepted manifest, capability, grant, lifecycle, and command-registry
  rules, with no special privilege.

Forbidden and refused:

- no private first-party bypass: a third-party plugin must be able to reach the
  same public surfaces; there is no Beacon-only channel;
- no raw PTY, GPU, or window handle, and no raw PTY access, input injection,
  grid mutation, or cross-plugin VM access;
- no input hot-path callback: transient input capture stays Core-owned,
  transient, bounded, revocable, and fail-open, and never runs a plugin
  callback on the input hot path;
- no Event-Bus exposure of target or annotation internals, and no cross-plugin
  target-metadata observation by default;
- no direct call into the compositor, terminal, or panel lifecycle; a selected
  target becomes a typed command in the accepted registry and nothing else.

The exact API spellings, capability identifiers, and version come from `W-29`
and `W-120`. This page decides none of them.

## Capability and trust boundaries

- Registration grants no authority: contributing a provider or target adds
  addressability only, and a command still executes under its own owner's
  grants with its own argument validation and budgets.
- Deny by default: the plugin requests no allow-all identifier, and unknown
  capability identifiers fail validation. The concrete identifiers are not
  defined here.
- Beacon observes only the public target metadata a provider supplies and never
  private plugin state or another plugin's internals.
- The session is a command namespace consumed by the keymap router, not a
  parallel input path; non-label keys fail open.

## Failure behavior

- Absent or disabled plugin: policy is absent, the Core mechanism is intact,
  and `bitty --safe` starts with zero third-party plugins.
- Denied capability: the gated call fails closed with a typed denial before any
  side effect, and no provider or session is created.
- Stale or unknown handle or label: the Core mechanism fails closed with
  `StaleTarget` or an expired-label error; Beacon never falls back to a default
  target or a silent dispatch.
- Provider fault: the faulty provider's targets are absent and the session
  continues with the remaining providers.
- Plugin runtime error or unload mid-session: the plugin host isolates the
  failure, the session ends, capture is revoked, labels are cleared, and the
  mechanism remains intact.

## Security review

Beacon crosses the plugin trust boundary, the capability model, safe mode, and
the input and dispatch paths. Independent security review is required before
any promotion beyond candidate. The reviewer must confirm that no security
enforcement point moves into the optional plugin and that no P0 control is
weakened:

- the Core mechanism stays capability-independent and always available; safe
  mode keeps zero third-party plugins with the mechanism functional;
- the plugin uses only the public capability-gated API, with no private
  first-party bypass and no raw PTY, GPU, or window handle;
- transient input capture stays capability-gated, transient, bounded,
  revocable, and Core-owned, with no plugin callback on the input hot path;
- the command-dispatch bridge routes through the accepted command registry and
  revalidates handles before returning a typed command;
- target and annotation internals are not exposed through the Event Bus, and no
  plugin receives another's target metadata beyond the accepted scope;
- public metadata only, no hot-path execution, and fail-closed behavior on
  stale handles, invalid labels, malformed metadata, and provider faults.

## Verification plan

Any later implementation must produce the evidence listed in
[evidence.md](evidence.md), at minimum:

1. Mechanism without the plugin: targeting works with zero plugins and in
   `bitty --safe`.
2. No private bypass: an equivalent third-party plugin can reach the same
   public surfaces.
3. Capability gating: a denied grant fails closed with a typed denial before
   any side effect.
4. No hot path: no plugin callback runs on the input hot path, and capture is
   bounded, revocable, and fail-open.
5. Fail-closed handles: a target invalidated after labeling cannot dispatch;
   generation checks reject stale handles.
6. Policy optionality: disabling or removing the plugin removes policy only.
7. Documentation gates: the repository-local `just check` passes with zero
   issues.

No verification result is claimed on this page.

## Alternatives considered

| Alternative                                                  | Trade-off                                                                                              | Disposition                                                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| Keep Beacon policy in Core                                   | Fewer moving parts, but couples a replaceable vocabulary and menu to the always-available mechanism.   | Rejected by ADR 0018; policy moves to the optional plugin.                              |
| Give the first-party plugin a private targeting path         | Convenient for the reference plugin, but creates a privileged channel a third-party plugin cannot use. | Rejected; the bootstrap fence forbids a private first-party bypass.                     |
| Let the plugin draw its own labels or capture input directly | Flexible presentation, but creates a privileged path and a hot-path callback.                          | Rejected; one Core-rendered annotation layer and Core-owned capture.                    |
| Make the plugin required and load it by default in safe mode | Always-available policy, but makes a fundamental capability depend on an optional plugin.              | Rejected by ADR 0018; the mechanism works with zero plugins and the plugin is optional. |

## Related

- [Beacon plugin documentation](README.md)
- [Schemas and contracts](schemas.md)
- [Evidence and links](evidence.md)
- [Beacon Targeting Framework (Candidate)](../../../specifications/beacon-targeting-framework-candidate.md)
- [Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md)
- [ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
- [Plugin system](../../../extensibility/plugin-system.md) (draft)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
