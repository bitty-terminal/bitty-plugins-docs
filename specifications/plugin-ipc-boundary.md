---
title: Plugin IPC Boundary
description: Research-derived design input for out-of-process plugins, the plugin event bus, and a unified Lua/IPC/CLI capability model from research record 041
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: true
sidebar_order: 42
---

# Plugin IPC Boundary

> Status: **draft**, research-derived design input recording the plugin-relevant
> conclusions of the workspace `research` record `origin/041.md` (lines 1-821).
> The record proposes IPC as a **second extension boundary** alongside the
> in-process Lua plugin API. This page is **not** an accepted contract, an RFC,
> or an implementation claim.

Accepted baselines already exist for the local IPC wire, auth, and scopes in
the [IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md)
(OQ-018, accepted 2026-08-29). That RFC's candidate addendum already covers the
overlapping workspace-automation and prior-art rationale, so this page records
only the plugin-facing consequences and cites the accepted document instead of
restating its wire contract. Where a statement aligns with an accepted
document, this page links it — relative for documents in this corpus, absolute
for sibling repositories; every other conclusion below is a **proposal or
observation from 041**, with source line ranges.

## 1. Purpose, status, and attribution

- The source record is read-only provenance in the workspace `research`
  repository; it was not edited or renamed for this page.
- `origin/041.md` is a single-pass record; no repeated conversation segments
  were found, so every cited range refers to unique content. The original is in
  Chinese, which `origin/` permits; this page records the conclusions in
  English.
- The record's frame is that the Lua plugin API and an IPC boundary **solve
  different problems and do not replace each other** (041.md lines 1-26).
- Section 12 lists the decisions still required before any conclusion here
  could become contract.

## 2. The second extension boundary: out-of-process plugins

Proposal (041.md lines 28-75): plugins need not run inside the Bitty process.
Over a local socket boundary they could be written in any language (the record
lists Rust, Python, Go, TypeScript, Java, and Shell) without adopting the Lua
runtime. The recorded consequence is a positioning shift from "Lua-extensible
terminal" to "terminal platform with a unified control protocol" (041.md lines
67-75) — a candidate framing, not an accepted product claim. Any external
process is still untrusted until an explicit scoped policy grants it
capabilities, per the accepted trust-boundary language in the
[IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md).

The record draws the suitability line explicitly (041.md lines 699-763):

| Surface                     | Recorded fit                                                                                                                    | Recorded reasons                                                                                                       |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| In-process Lua plugin       | keybinding, theme, render hook, UI decoration, statusline, layout behavior, small commands, event handlers                      | low latency, high call frequency, tight UI/Core coupling                                                               |
| Out-of-process (IPC) plugin | AI, Git daemon, language tooling, indexer, sync, database, network service, large computation, external application integration | independent lifecycle, may crash, complex dependencies, other languages, network/database use, coarse call granularity |

Alignment: the accepted
[Isolation and Resource RFC](isolation-resource-rfc.md) already names "a helper
process with scoped IPC" as a high-isolation extension direction, and the draft
[Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md) Layer 4
defines declared, digest-pinned native helper processes over stdio or a
host-owned local channel (post-1.0). 041's direction is broader than Layer 4:
it makes arbitrary external processes plugin participants rather than
manifest-declared helpers. Accepting that needs a process lifecycle and
supervision contract first, which no current document defines (candidate).

## 3. Core minimization: heavy plugins as separate processes

Proposal (041.md lines 79-148): heavy surfaces — provider, agent runtime, MCP,
embedding, memory, SQLite, network, and context management — should not be
embedded into Core. A separate `bitty-ai` daemon would keep Core from growing
HTTP, TLS, SQLite, AI SDK, embedding, and MCP dependencies; the record's sketch
has Core knowing only coarse verbs such as `agent.spawn`, `agent.send`,
`agent.cancel`, `panel.attach`, `panel.output`, and `panel.close` (041.md lines
124-146). The record connects this to the earlier position that `bitty-ai`'s
network dependency stays independent of Core (041.md line 148).

The recorded verbs are illustrative method names, not an accepted registry.
The dependency-minimization direction matches the "no embed third-party crate
bloat" rule and helper-process staging stated in the draft
[Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md), and the
isolation direction in the accepted
[Isolation and Resource RFC](isolation-resource-rfc.md); the daemon split
itself remains a proposal.

## 4. Panel and Agent as public protocol surfaces

Proposal (041.md lines 152-282): a public Panel protocol (`panel.create`,
`panel.write`, `panel.focus`, `panel.move`, `panel.resize`, `panel.close`) would
let plugins operate Panels without touching internal Rust structs; the record
sketches CLI and JSON forms (041.md lines 171-200) and a
`Plugin -> Bitty IPC Protocol -> Panel Manager` layering (041.md lines 206-220).

The record's driving example is that **Agent and Panel lifecycles are not
bound**: an Agent attaches to a headless Panel, leaves it, and later attaches to
another, using verbs such as `panel.list`, `panel.inspect`, `panel.attach`,
`panel.detach`, `panel.send_input`, and `panel.read_output` instead of a
`&mut Panel` handle (041.md lines 224-282).

These Panel method names are **candidate and unaccepted**. Panel semantics are
owned by the accepted Panel Runtime RFC in the sibling `bitty-terminal-docs`
repository, whose provider and ecosystem surface remains open (`RFC-OQ-1`
through `RFC-OQ-9`), and the Panel-as-host direction is already recorded in the
[Plugin Ecosystem Model](plugin-ecosystem-model.md) section 9.
This page records only the IPC consequence: a public protocol is the mechanism
that would keep plugin and Agent integrations off internal types.

## 5. Plugin-to-plugin communication and the event bus

Proposal (041.md lines 286-367): the IPC surface need not be only plugin to
Core; an event bus could carry plugin-to-plugin events. Recorded examples are
`git.branch.changed` to a statusline plugin, `command.completed` from a panel
history plugin to an AI plugin, and `agent.status.changed` from `bitty-ai` to a
dashboard plugin (041.md lines 304-332). The record's candidate event taxonomy
includes panel created/closed/focused, command started/finished, cwd and
environment changes, agent started/finished, and workspace changes (041.md
lines 336-351). The recorded intent is that in-process Lua subscribers
(`bitty.on(...)`) and external subscribers (`subscribe(...)`) share one event
semantics (041.md lines 353-367).

The accepted [Plugin Platform RFC](plugin-platform-rfc.md) already defines the
in-host event pipeline with classes, budgets, and fail-open rules;
cross-process subscription scope, delivery guarantees, backpressure, and
authorization for the candidate bus are not defined by any accepted document
and would have to reuse the accepted IPC rate-limit and budget contracts rather
than relax them (candidate, open item 4).

## 6. One capability model, three frontends

Proposal (041.md lines 371-446 and 765-821): the Lua API, an IPC binding, and a
CLI control surface should be three frontends of one capability model rather
than three parallel systems. The record lists candidate capability names —
`panel.list`, `panel.create`, `panel.focus`, `panel.close`, `workspace.list`,
`workspace.switch`, `terminal.send_input`, `terminal.read_history`,
`command.run`, `notification.send` (041.md lines 394-409) — and shows the same
action expressed as a Lua call, a JSON request, and a CLI command (041.md lines
411-434). It closes with a candidate "Bitty Capability Protocol" naming for the
capability layer and an illustrative domain list: Panel, Workspace, Terminal,
Command, Agent, Plugin, Notification, Event, Clipboard, and History (041.md
lines 795-818).

Alignment: the accepted corpus already routes registries so CLI, palette, IPC,
and Agents reuse one surface (see the [Plugin Platform RFC](plugin-platform-rfc.md)
and the [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md),
which declares `result_schema` for CLI, palette, IPC, and Agent reuse). The
unaccepted addition is an external-process binding that consumes the same
capability registry; the method names and domains above are candidate
vocabulary only.

## 7. Capability tokens and permission display (unaccepted)

Proposal (041.md lines 450-507): an external plugin would authenticate to the
IPC server and receive a capability token; Core would evaluate per-capability
allow/deny, and a `bitty plugin permissions <id>` command could display granted
and denied capabilities. The record sketches a manifest shape (041.md lines
456-464):

```toml
[permissions]
panel.read = true
panel.create = true
terminal.input = false
filesystem.read = false
network = false
agent.control = false
```

This sketch is **unaccepted and diverges from the accepted capability
grammar**: the accepted model uses closed, owner-qualified identifiers with
parameters (for example `terminal.semantic-read`, `process.spawn:git`),
deny-by-default grants bound to plugin identity and manifest hash, and no
wildcards ([Plugin Platform RFC](plugin-platform-rfc.md);
[Isolation and Resource RFC](isolation-resource-rfc.md)). The boolean
`[permissions]` table must not be read as schema, and a future plugin process
would receive scoped grants, never ambient authority; mapping a capability
token to the accepted IPC scopes and plugin grants is an open item (open item
3). The record's security argument — a scoped protocol boundary is easier to
defend than exposing the whole Lua Core API to third parties — is consistent
with the accepted deny-by-default posture.

## 8. Failure isolation, supervision, and debugging

Proposal (041.md lines 511-556): an out-of-process plugin that crashes must not
take down Bitty; the record poses restart, disable, and log-surfacing options
rather than defining a policy. Alignment: resource isolation and failure
semantics for IPC/MCP clients are accepted in the
[Isolation and Resource RFC](isolation-resource-rfc.md), but no accepted
document defines a plugin-process supervisor, restart policy, or reconnection
semantics (candidate, open item 2).

Proposal (041.md lines 560-607): candidate inspection surfaces such as
`bitty msg panel list`, `bitty msg tree`, and `bitty msg events` would expose
panel ownership, workspace hierarchy, and an event trace; the record values
them for a future DevTools. Accepted baseline: the
[IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md)
normatively defines instance discovery and selection, including the
`bitty ctl instance list` command; its own CLI document and the broader
`bitty msg`/`bittyctl` method surface are marked candidate there (RFC lines 88
and 720), so the `bitty msg panel list`/`tree`/`events` shapes here are
candidate only.

## 9. Controlling a running instance and multi-instance addressing

Proposal (041.md lines 611-656): a `bittyctl`-style client would be a thin IPC
client, not a second implementation of Bitty features, mirroring `hyprctl` for
a running compositor. Accepted baseline: the
[IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md)
defines instance discovery and selection for exactly this use; `BITTY_SOCKET`
and instance identifiers stay advisory and never credentials.

Proposal (041.md lines 660-695): several running instances would expose one
socket each under the user runtime directory, and an address chain
`instance -> workspace -> panel` would let plugins or Agents target a specific
surface; the record sketches a `bitty://instance/<n>/workspace/<n>/panel/<n>`
form (041.md lines 691-693). Multi-window and daemon modes remain deferred by
[ADR 0008](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0008-headless.md)
in the shared governance corpus, and the URI form is a candidate illustration,
not an accepted grammar (open item 5).

## 10. Candidate three-layer extension model

Proposal (041.md lines 765-821): Core plus a Lua plugin API and an IPC API, with
a capability layer above exposing Lua, socket, and CLI frontends. This is
compatible with the "runtime flat, semantics layered" principle recorded from
`origin/040.md` in the [Plugin Ecosystem Model](plugin-ecosystem-model.md)
section 3: the IPC peers are still flat runtime peers managed through one
plugin model, not nested runtimes. The combined three-layer framing is
**candidate design input**, not an accepted architecture.

## 11. Mapping to the existing corpus

| Theme                                          | Existing document                                                                                                                                                       | Relationship                                                                                         |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Lua versus out-of-process plugin suitability   | [Isolation and Resource RFC](isolation-resource-rfc.md), [Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md)                                         | Aligns with the accepted helper-process direction; broader IPC plugin participants are candidate     |
| Core minimization and `bitty-ai` daemon split  | [Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md)                                                                                                  | Aligns with the draft no-embed rule and Layer 4 staging; the daemon split is candidate               |
| Panel/Agent protocol surface                   | [Plugin Ecosystem Model](plugin-ecosystem-model.md) section 9, sibling Panel Runtime RFC                                                                                | Extends; Panel ownership and provider surface stay with the sibling contract                         |
| Plugin-to-plugin event bus                     | [Plugin Platform RFC](plugin-platform-rfc.md)                                                                                                                           | Extends the accepted event pipeline to cross-process subscribers; candidate                          |
| Unified capability model and method vocabulary | [Plugin Platform RFC](plugin-platform-rfc.md), [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md)                                                        | Aligns on one registry for CLI/palette/IPC/Agent reuse; the external binding and names are candidate |
| Capability tokens and `[permissions]` sketch   | [Plugin Platform RFC](plugin-platform-rfc.md), [Isolation and Resource RFC](isolation-resource-rfc.md)                                                                  | Diverges from the accepted capability grammar; must be reconciled, not added in parallel             |
| Crash isolation and supervision                | [Isolation and Resource RFC](isolation-resource-rfc.md), [IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md) | Aligns on untrusted-client boundaries; supervisor semantics are unaddressed                          |
| Control CLI and multi-instance addressing      | [IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md)                                                          | Aligns with accepted instance selection; `bittyctl` verbs and `bitty://` addressing are candidate    |
| Three-layer extension framing                  | [Plugin Ecosystem Model](plugin-ecosystem-model.md) section 3                                                                                                           | Consistent with "runtime flat, semantics layered"; combined framing is candidate                     |

## 12. Research open items

These are **research open items, not accepted open questions**. Each must be
decided in the owning contract before any conclusion here becomes contract:

1. Ownership of the capability/method vocabulary: which repository owns a
   shared method registry across Lua, IPC, and CLI, and how names are versioned.
2. Plugin-process lifecycle and supervision: launch, restart, disable, log
   surfacing, reconnection, and crash containment semantics for out-of-process
   plugins.
3. Capability-token mapping: how a token maps to the accepted IPC scopes and
   per-plugin grants without creating a parallel permission system.
4. Cross-process event bus semantics: subscription authorization, delivery
   guarantees, backpressure, and interaction with the accepted event pipeline
   budgets.
5. Multi-instance addressing grammar: whether any `bitty://`-style address is
   adopted, and how it relates to accepted instance selection and the deferred
   daemon mode.
6. Panel protocol ownership: how candidate Panel verbs interoperate with the
   sibling Panel Runtime RFC open questions (`RFC-OQ-1` through `RFC-OQ-9`)
   without splitting ownership.
7. Whether external processes are a distinct plugin class in the manifest and
   package model or a transport option of the existing plugin model.
