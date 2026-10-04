---
title: Plugin history and storage policy
description: Plugin-facing policy for history and storage ownership privacy retention capabilities and per-plugin page sets
category: extensibility
audience: plugin-author
document_type: specification
status: accepted
website_publish: true
sidebar_order: 21
---

# Plugin history and storage policy

## Document status

Accepted. This document is the `W-137` deliverable for the plugin-facing history
and storage policy that sits above the Core storage boundary. That boundary -
the reconciliation of the four storage objects (segmented transcript, command
history, session snapshots, and per-plugin key-value state) - is owned by
`W-131` and now accepted; this page specializes it for plugin authors and
must not weaken it. This document accepts the plugin-facing boundary decision;
it does not accept downstream SDK/Core/plugin contracts, does not authorize
implementation, and does not describe implemented behavior. Every ownership,
privacy, and retention statement here is decided direction for plugins;
downstream spellings and mechanics remain candidate per Open points.
Frontmatter `status` is `accepted` per the repository metadata schema.

- Owning task: `W-137` promotion (bitty-plugins-docs), CarryCtx `CTX-0075`,
  Issue
  [#137](https://github.com/bitty-terminal/bitty-plugins-docs/issues/137).
  Created by `W-137` (CarryCtx `CTX-0072`, Issue #131); see References.
- Upstream reconciliation:
  [Storage and History Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/storage-and-history-boundary.md)
  (`W-131`, accepted), grounded in
  [ADR 0016](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md)
  Boundary 4 and binding constraints 8 and 9.
- Downstream owners named but not decided here: `W-139`
  (bitty-plugin-sdk, the accepted public history, storage, search, and
  selection APIs), `W-146` (bitty, Core integration), and the `history` plugin
  package, whose own public-API history policy is its `CTX-0003`.

## Purpose and scope

This policy fixes the plugin-visible rules for history and storage so that a
first-party or third-party history plugin builds on the same public,
capability-gated surface with no private bypass. It answers four questions for
a plugin author: which storage objects a plugin may touch, how privacy and
retention work, how an external history system is reached, and what a
history/storage plugin may declare and call.

In scope:

- object ownership for plugins across the four storage objects;
- privacy, capture opt-in, retention, deletion, purge, and export semantics;
- the external-history (for example Atuin) integration boundary;
- the capability declarations a history or storage plugin requests, and the
  deny-by-default posture around them;
- high-level requirements on the public history and storage API surface;
- the standard per-plugin documentation page set for the history keeper, as
  real content guidance.

Out of scope and not decided here:

- exact API spellings, capability identifiers, schemas, storage formats,
  segment seal bounds, and retention defaults, which belong to `W-139` and
  `W-146`;
- the Core object reconciliation, ownership split, and extracted-storage
  mechanics, which belong to `W-131`;
- the AI session, context, and memory export model, which is owned on the
  `bitty-ai-docs` side and composes with, but is not redefined by, this policy;
- any implementation, migration, or repository creation.

Nothing here weakens a normative security control. Where a control or threshold
appears to need change, it is recorded under "Open points" instead.

## Normative sources this specification must not weaken

This policy must be read together with, and must not weaken:

- The [security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md),
  the [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md),
  the [risk register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md),
  and the [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md).
  The controls that bind this policy include plugin capability checking with
  least privilege (`P0-AC-012`), resource budgets (`P0-AC-014`), hot-path
  exclusion (`P0-AC-015`), Terminal Truth ownership (`P0-AC-016`), trace
  minimization and redaction with user-only files and export-preview equality
  (`P0-AC-026`), secret-storage tiers (`P0-AC-036`), the panel lease write gate
  (`P0-AC-039`), and no shell-string construction or interpolation on any
  external invocation (`P0-AC-009`); argv-first validated arguments are this
  policy's mechanism for that control.
- The [Storage and History Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/storage-and-history-boundary.md)
  (`W-131`, draft): the four objects stay distinct, no universal database is
  authorized, raw stdout is not persisted by default, and there is no private
  first-party bypass.
- The accepted [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md),
  the [Plugin Platform RFC](../specifications/plugin-platform-rfc.md), the
  [Plugin Host Runtime RFC](../runtime/plugin-host-runtime-rfc.md), and the
  Isolation and Resource RFC. The
  published `bitty.store` ceilings and the atomic-commit rule stay
  authoritative.
- The Terminal Truth and Panel Runtime invariants in `bitty-terminal-docs`: a
  plugin may alter presentation, never Terminal Truth, and persistence must
  never replace or mutate volatile scrollback.
- The accepted [IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md):
  agent access is read-only by default, and terminal output is observation
  data, never instructions.

## Terminology

- **Core**: the always-available terminal mechanism that works with zero
  plugins and in `bitty --safe`; it owns Terminal Truth, the event source, the
  capability gate, and resource budgets.
- **History keeper**: the official plugin that owns persistence policy above
  the Core event and storage-capability surfaces. It is not yet accepted and
  not implemented.
- **Segmented transcript**: the append-only, segment-sealed log of ordered
  panel and terminal events and output, from which command history and exports
  are derived; the canonical byte log, distinct from any searchable index.
- **Command history**: the small, indexable record of commands - command text,
  working directory, timestamps, duration, exit code, actor, and an optional
  external reference - retained separately from raw output.
- **Session snapshot**: the Core-owned, versioned save/restore capture of
  workspace and pane state. It is not the live terminal snapshot bridge read.
- **Per-plugin KV**: the plugin-scoped, quota-bounded persistent key-value
  state exposed as `bitty.store`, distinct from `bitty.settings` and granting
  no filesystem authority.
- **Capture opt-in**: an explicit, per-user decision that enables a persistence
  path; the default is off for anything derived from raw terminal output.
- **Purge**: an explicit deletion that removes bytes, not merely a query
  filter.
- **Page set**: the standard per-plugin documentation partition (README,
  design, schemas, evidence) defined by the
  plugin documentation index.
- **Candidate**: a proposal that is not decided; candidate status is not
  acceptance and is not implementation.

## Object ownership for plugins

A plugin never opens a store file, a database, or a native library directly.
Access is always mediated, and the four objects stay distinct: none may become
a universal database or a default sink for raw terminal output.

| Storage object       | Plugin access mode                                                                             | Mediated path                                                                                                     | Core-only or forbidden to plugins                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Per-plugin KV        | Direct, plugin-authored reads and writes for the plugin's own keys only.                       | `bitty.store.get` / `bitty.store.set`, synchronous and quota-bounded in the plugin's own namespace.               | No cross-plugin read, no filesystem path, no terminal-derived content as a shim.                     |
| Segmented transcript | Policy ownership above a mediated append and scoped read surface; never a raw byte handle.     | Core event stream plus a capability-gated storage surface whose exact spelling belongs to `W-139`.                | No PTY hook, no shell-integration parsing, no guessed command boundaries, no direct segment access.  |
| Command history      | Policy ownership above a mediated, scoped query and read surface; never a database connection. | Core event stream plus a capability-gated history read surface; a rebuildable index is never the store of record. | No SQL or store internals, no cross-panel read without matching scope, no external private database. |
| Session snapshots    | No plugin access.                                                                              | None. Core save/restore is a mechanism that must work with zero plugins and in safe mode.                         | Read and write are Core-only; a plugin may never gate, read, or mutate a snapshot.                   |

Ownership rules that follow from the table:

- **Per-plugin KV is the plugin's own bounded sandbox surface.** It is intended
  for plugin state, not for terminal-derived sensitive content, and the plugin
  is responsible for not persisting unredacted secrets in it. Its bounds are
  the published `bitty.store` ceilings: 256 KiB total per plugin, 8 KiB per
  value, depth 8, and 1024 nodes, with atomic commits and no partial writes.
- **Transcript and history are Core-sourced.** Core observes PTY output, input,
  working directory, process lifecycle, and exit codes, and can emit events on
  the event bus; the history keeper persists above that surface. `OSC
133` markers are an advisory boundary signal only - never proof of a command
  or of a working directory.
- **Session snapshots are Core-only.** Save and restore must remain available
  without any plugin, so an optional plugin is never in that path.
- **No direct database or filesystem access.** A plugin may not connect to a
  database, open a segment file, or link a native storage library. Neither
  `bitty.store` nor `bitty.settings` grants a path or a handle, and Plugin API
  v1 defines no Lua `fs.*` entry point.

## Privacy and retention

Privacy is a default, not a setting a plugin opts the user into.

- **Opt-in capture.** Nothing derived from raw terminal output is persisted by a
  plugin without an explicit, per-user capture opt-in. Installing a history
  plugin does not by itself enable capture; the plugin must state what it
  captures and the user must consent. This is the plugin-owned persistence
  rule; the one accepted Core exception is the bounded session snapshot Core
  writes on normal interactive exit (see the Core storage boundary), which is
  Core-owned and outside plugin control.
- **Raw stdout is not persisted by default.** The command tier is the least
  sensitive tier; full output and full-replay tiers are separate, larger, and
  opt-in only. Recording input is a further, distinct opt-in and is never
  implied by output capture.
- **No-echo input never persists.** Input while a target PTY is in no-echo mode
  never enters a grid, scrollback, snapshot, transcript, trace, or agent
  observation.
- **Secret-minimizing.** A plugin must not persist unredacted secrets, and
  terminal-derived sensitive content travels only through the opt-in history
  path, never through per-plugin KV as a shim. Redaction composes with the
  secret-tier rules and typed redaction from the start.
- **User-only files.** Persisted history and store files carry user-only
  permissions, consistent with the trace-minimization control.
- **Retention by age, size, and count.** Retention is bounded and stated in
  user-facing terms. Sealed segments are deleted under the age and size policy;
  command records carry count, age, and byte bounds, with the command-only tier
  as the least sensitive default.
- **Purge removes bytes.** An explicit purge is authoritative: it removes the
  original bytes and any derived index entries, and a post-purge export or
  query returns nothing. A rebuildable index may never resurrect deleted
  content.
- **Deletion and uninstall semantics.** Per-plugin KV entries are deleted only
  by uninstall, an explicit `nil` write, or an explicit user purge. Removing a
  plugin should preserve retained history by default and report where it
  remains; full purge is a separate, explicit choice. A retained history
  object is not deleted merely because a panel closed.
- **Export semantics.** Export and query return redacted and truncated records
  with attribution, and an export preview equals the actual export
  byte-for-byte. Deletion and export are distinct operations; a plugin must not
  describe a filtered view as a purge.
- **Third-party and first-party parity.** The official history keeper obeys
  the same opt-in, retention, purge, and export rules as any third-party
  plugin.

## External history integration

The history keeper does not rebuild an external shell-history system such as
Atuin, and it never reads that system's private database.

- **Supported interface only.** An external history system is reached only
  through its documented CLI or API. Reading `history.db` or any other private
  schema directly is forbidden, because the internal schema is not a public
  contract.
- **Argv-first, validated arguments.** Every external invocation is argv-first
  with validated arguments and no shell-string construction or interpolation,
  satisfying the no-shell rule of `P0-AC-009`. A missing or unsupported CLI or
  API fails closed with a diagnostic rather than falling back to file access.
- **Provider, importer, and sink modes.** The candidate integration modes are
  provider (external history surfaced live into a Bitty history UI), importer
  (external records mapped into command history), and sink (Bitty-recorded
  commands written back through the supported interface). The exact mode set
  and merge semantics remain candidate.
- **No raw output leaves Bitty.** Only the small command record may cross the
  boundary; raw panel output never enters an external history system.
- **Opaque external reference.** A command record may carry an opaque
  reference to an external system. The reference does not make that system's
  schema public, and a dangling reference after provider removal or expiry
  surfaces as typed unavailability, never as a silent gap.

## Capabilities

Host access is granted by capability, with least privilege as the default and
no allow-all boolean anywhere.

- **Deny-by-default.** A history or storage plugin reaches no host surface
  without a specific grant. The capability layer restricts host APIs; it is not
  a complete sandbox, and native in-process plugins remain outside the accepted
  security model.
- **Parameterized, narrow grants.** Capability identifiers carry the scope they
  apply to (for example a directory scope, a panel or workspace scope, or a set
  of allowed external commands) rather than a wildcard. There is no wildcard
  default, and a call outside the declared scope fails closed.
- **No widening.** A plugin cannot widen its own grant at runtime. A capability
  increase blocks an update pending explicit user review, reusing the package
  manager capability-increase gate.
- **No private first-party bypass.** The official history keeper uses the same
  public, capability-gated API as a third-party plugin. No first-party
  component may claim an internal storage path, a private database handle, or
  an ungated capability.
- **Candidate capability categories.** The categories a history or storage
  plugin is expected to request are a scoped history or transcript access
  capability, terminal semantic read where needed, process or external-CLI
  access for a validated external-history interface, and environment read where
  a key is explicitly declared. The exact identifiers and their grammar are
  parked to `W-139` as the owner of the accepted surface; this policy fixes
  only the deny-by-default, parameterized, no-widening, no-bypass rules.
- **No v1 filesystem shortcut.** Plugin API v1 defines no Lua `fs.*` entry
  point, so a storage-capability shape that exposes sandboxed directory access
  is future contract work, not a v1 grant, and it may not be reintroduced as a
  private path.

## Public API requirements

The public history and storage surface is the only path, and its exact
spellings belong to `W-139`. At the policy level, the surface must satisfy:

- **Explicit scope and bound on every read.** List, query, get-output, and tail
  operations carry an explicit scope (panel, workspace, directory, or global)
  and an explicit result bound. A read without a matching scope is denied
  fail-closed, and a scan or export cannot be unbounded.
- **Redacted and truncated records with attribution.** Returned records are
  redacted and truncated where policy requires, and each carries attribution
  (panel, workspace, command, actor, and timing as applicable) so a consumer
  can tell what it is reading and from where.
- **No SQL or store internals.** The surface never exposes SQL, a query
  language over internal tables, a store file path, or backend-specific
  internals. Which backend serves the data is not plugin-visible.
- **Event and capability surfaces, not a database.** Core keeps the event
  generation and the storage-capability gate; the persistence policy lives
  above it. A rebuildable index is optional and never authoritative.
- **Read-only default for agents.** Agent consumption is read-only by default,
  never owns history, and treats returned output as observation data rather
  than instructions.

## Page sets

A documented history plugin uses the standard per-plugin page set as real
content, not placeholders. Create a page only when it has real content.

- **README.** Identity (owner-qualified plugin id), current stage, owning
  repository, declared capabilities, lazy triggers, and links to the other
  pages. State the capture posture (opt-in or not) and the tiers the plugin
  offers.
- **Design.** The scope and the object-ownership mapping: which of the four
  objects the plugin touches, how it uses the Core event and storage-capability
  surfaces, its retention and purge model, and its external-history integration
  mode. Record the privacy posture and the denied paths it deliberately does
  not take.
- **Schemas.** The manifest capability declarations and any command-record or
  configuration schema the plugin owns, with the bounds it applies. Exact
  public API spellings remain with `W-139` and are linked, not copied.
- **Evidence.** The verification evidence the plugin will produce for capture
  opt-in, default-persistence privacy, purge-removes-bytes, scope-escape
  denial, quota atomicity, and external-argument validation. Until that
  evidence exists, the plugin is candidate or pre-implementation and the pages
  must say so.

The `history` plugin package is the concrete owner of this page set; its
public-API history policy is carried by its own `CTX-0003`. This policy fixes
what the page set must cover and does not decide the plugin's content.

## Security review

Independent security review sign-off is recorded for this promotion (boundary-decision scope; verification evidence stays with `W-139`/`W-146`). The
reviewers confirmed that no object is persisted by default when it carries
sensitive terminal-derived content, that capture and input recording stay
sensitive terminal-derived content, that capture and input recording stay
separate opt-ins, that per-plugin KV is not used as a shim for terminal output
or secrets, that no plugin-reachable path opens a database, segment file, or
native library, that external history access is argv-first and validated with
no shell construction, and that no first-party bypass or capability widening is
introduced. The downstream owners (`W-139` for the SDK surface and `W-146` for
Core integration) each require security review again before their own merge.

## Verification plan

This is a policy specification with no executable verification of its own. Any
later implementation of the history keeper must prove, at minimum:

1. **Opt-in capture holds.** With no opt-in, no transcript, command-history, or
   raw-output bytes are written; enabling capture is explicit, and recording
   input is a separate opt-in.
2. **Sensitive output is not persisted by default.** A seeded-secret corpus
   never appears in default persisted output, and no-echo input never reaches a
   store, snapshot, or trace.
3. **Purge removes data.** Purging or expiring a record removes the original
   bytes and any derived index entry, and a rebuildable index cannot resurrect
   deleted content.
4. **Scope escape is denied.** A cross-panel, cross-workspace, cross-plugin, or
   out-of-directory read without the matching scope is denied fail-closed.
5. **Budgets and denials are atomic.** Per-plugin KV and retention bounds hold
   exactly, and an over-budget write leaves the previous state intact with no
   eviction.
6. **Capability rules hold.** Every ungranted call denies, grants are
   parameterized and not widenable at runtime, and an official history plugin
   passes the same denial suite as a third-party plugin.
7. **External arguments are validated.** Every external-history invocation is
   argv-first with validated arguments and no shell-string construction or
   interpolation.
8. **Safe mode stays clean.** `bitty --safe` starts with zero third-party
   plugins and reads no transcript, command history, session snapshot, or
   per-plugin KV.
9. **No private first-party bypass.** The official history keeper uses the same
   public, capability-gated API as any third-party plugin.
10. **Documentation gates pass.** The repository-local `just check` passes with
    zero issues.

## Alternatives considered

| Alternative                                                          | Disposition                                                                                                                                                  |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Let a history plugin open the transcript segments or a database file | Rejected: it breaks the mediated-access rule and the no-direct-filesystem boundary; Core exposes event and storage-capability surfaces instead.              |
| Persist raw stdout by default                                        | Rejected: it violates the secret-minimizing default and the opt-in requirement; raw output is a separate, bounded, opt-in tier.                              |
| Treat per-plugin KV as the history store                             | Rejected: it bypasses the opt-in history path and the capability model, and KV is not a sink for terminal-derived content.                                   |
| Read an external history system's private database                   | Rejected: the internal schema is not a public contract; only the supported CLI or API is allowed, argv-first and validated.                                  |
| Give the official history keeper a private storage handle            | Rejected: no private first-party bypass; first-party and third-party plugins use the same public API.                                                        |
| Let a storage capability expose a raw filesystem path in v1          | Rejected as a v1 surface: Plugin API v1 defines no Lua `fs.*` entry point; a sandboxed storage shape is future contract work and must stay capability-gated. |
| Read or write session snapshots from the history plugin              | Rejected: save/restore is a Core mechanism that must work with zero plugins and in safe mode.                                                                |

## Affected contracts

- [Storage and History Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/storage-and-history-boundary.md)
  (`W-131`, draft): the Core object reconciliation this policy specializes; the
  four objects stay distinct.
- [ADR 0016](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md)
  (Boundary 4; binding constraints 8 and 9): history is opt-in,
  secret-minimizing, and bounded, and external history uses the supported
  interface only.
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) and
  the [`bitty.store` reference](../sdk/reference/store.md): the accepted store
  surface and its ceilings that this policy reuses.
- [Plugin system](plugin-system.md) and
  [Plugin package management](package-management.md): capability, lifecycle,
  and capability-increase review reused here.
- [History-provider direction](../packaging/plugin-reuse-and-providers.md):
  the candidate history-provider input this policy specializes for plugins.
- Plugin documentation index: where the standard
  page set is registered, and the
  plugin roadmap: the candidate history plugin
  entry.
- [Panel History (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-history-candidate.md)
  and the [Terminal State RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/terminal-state-rfc.md):
  the terminal-platform direction behind the object split; summarized here,
  owned there.
- [SDK surface owner](https://github.com/bitty-terminal/bitty-plugin-sdk)
  (`W-139`) and the `history` plugin package
  ([bitty-terminal/history](https://github.com/bitty-terminal/history), its
  `CTX-0003`): downstream owners; their content is not decided here.

## Open points

The following details are parked, not decided. Each park names the owning task
and the reason. None is a global open question: none blocks the current
milestone, and no implementation evidence forces one yet.

- **Capability identifiers and grammar** parked to `W-139`: exact history and
  storage capability identifiers, their scope-parameter shape, and the
  declaration syntax a history plugin uses. This policy fixes the rules, not
  the spellings. The `W-137` brief also names capability grammar for history
  and storage; where the two briefs overlap, the exact identifier/declaration
  spellings are resolved with `W-139` (the SDK surface owner), and that
  division is recorded here rather than left implicit.
- **Public API spellings and bounds** parked to `W-139`: the accepted history,
  storage, search, and selection API surface, including the concrete bound
  defaults for list, query, get-output, and tail.
- **Redaction format in persisted history** parked to `W-137` and `W-139`:
  typed redaction and export-preview equality must compose with the secret-tier
  rules from the start.
- **Cross-object correlation and dangling references** parked to `W-137`: the
  lifecycle of external history references after expiry, deletion, or provider
  removal, and any federated-query merge semantics.
- **Retention defaults and segment bounds** parked to `W-146`: exact age, size,
  and count defaults, and the sealed-segment bound, once the extracted storage
  mechanics are decided.
- **External-history transport** parked to `W-139` and the external system
  owner: the supported CLI or API shape for each mode, argv-first and validated,
  without a private database fallback.
- **Session snapshot and store mechanics placement** parked to `W-146` and
  `W-139`: whether session-snapshot or per-plugin KV mechanics move to the
  extracted storage component while Core retains restore re-derivation and the
  SDK retains the KV contract.

## Acceptance criteria

- Object ownership for plugins is explicit for all four storage objects, and
  session snapshots are Core-only with no plugin access.
- Privacy and retention are stated: opt-in capture, raw stdout not persisted by
  default, separate input recording, bounded retention, and purge that removes
  bytes.
- External history integration is limited to the supported CLI or API with
  argv-first validated arguments, with no private-database path.
- Capability rules are deny-by-default, parameterized, non-widening, and free
  of a private first-party bypass.
- Public API requirements state explicit scope and result bounds, redacted and
  truncated records with attribution, and no SQL or store internals, with exact
  spellings delegated to `W-139`.
- The page-set section gives real content guidance for README, design, schemas,
  and evidence, not placeholders.
- The Core storage boundary, the SDK surface, and the history plugin package
  are cross-linked without deciding their content.
- Policy boundary is accepted; downstream SDK/Core/plugin objects remain candidate, nothing is described as implemented, and no P0 control is weakened.
- The documentation index routes to this page, and `just check` passes with
  zero issues.

## P0 Review Sign-off

| Role                | Scope                                                                                   | Requirement                                                                       |
| ------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `category-owner`    | Object ownership, capability rules, and public-API policy correctness                   | Approve; confirms plugin ownership, no-bypass, and delegation to `W-139`.         |
| `security-reviewer` | Privacy default, capture opt-in, redaction, purge, capability, external-argument safety | Approve recorded for promotion; verification evidence stays with `W-139`/`W-146`. |
| `docs-curator`      | Taxonomy, metadata, links, terminology, and index synchronization                       | Approve; confirms discoverability, schema, and page-set guidance.                 |

## References

- Issue [#131](https://github.com/bitty-terminal/bitty-plugins-docs/issues/131)
  (CarryCtx `CTX-0072`, plan key `W-137`) and promotion Issue
  [#137](https://github.com/bitty-terminal/bitty-plugins-docs/issues/137)
  (CarryCtx `CTX-0075`).
- [Storage and History Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/storage-and-history-boundary.md)
  (`W-131`, accepted) and
  [ADR 0016](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md).
- [Security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md),
  [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md),
  and [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md).
- [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md)
  (accepted surface) and [`bitty.store` reference](../sdk/reference/store.md).
- [Panel History (Candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-history-candidate.md),
  [Terminal State RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/terminal-state-rfc.md),
  and the [IPC and Agent RFC](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/specifications/ipc-agent-rfc.md).
- Downstream owners: [bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk)
  (`W-139`) and the `history` plugin package
  ([bitty-terminal/history](https://github.com/bitty-terminal/history), its
  `CTX-0003`).
