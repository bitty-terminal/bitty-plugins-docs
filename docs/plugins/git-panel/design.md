---
title: Git panel plugin design
description: Scope tiled presentation and capability design for the git panel plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 47
---

# Git panel plugin design

## Document status

Current source at registry-pinned `4f54634f`. Core `b5145675` provides the
grant-gated git spawn extra with a host-resolved `cwd = "pane"` semantic, but
repository-metadata policy, isolation evidence, and panel usability remain
unproven.

## Purpose and scope

Users need git branch, status, diff, and log presentation inside the terminal
workspace without shelling out through unconstrained process authority.

## Normative sources this specification must not weaken

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and isolation. The accepted
[`tools.git` slice](../../../packaging/plugin-reuse-and-providers.md#accepted-toolsgit-contract-v1)
does not create a general stable process API.

## Terminology

Command data and declarative scenes presented in a focusable overlay are not a
host panel slot. An argv allowlist constrains verbs, and the `pane` cwd semantic
selects a starting directory; neither sandboxes the child nor closes repository
metadata.

## Current behavior

Current source composes git branch/status/diff/log command data over
allowlisted `process.spawn:git` with the manifest-declared `[tools.git]`
contract. The Lua policy is pure plus Layer 2 system-CLI reuse: the allowlist
permits only the `git` binary with the seven read-only verbs (`status`,
`diff`, `log`, `branch`, `show`, `rev-parse`, `ls-files`) under fail-closed
bounds (at most 32 args of at most 256 bytes each), and the working-tree read
scope is requested by a manifest-declared working-tree read pattern.

Out of scope: terminal truth, input, parser, and render hot paths; arbitrary
process spawn, shell interpolation, network, clipboard, terminal-input, or
persistent-state authority; and write verbs beyond the allowlisted read-only
set. Panel presentation uses the focusable overlay session
(`bitty.ui.overlay.acquire`/`update`) rather than a host panel slot, so the
plugin never registers a panel provider. A host-owned panel-provider lifecycle
remains an explicit non-goal of the current package.

## Mechanism and policy split

Core owns terminal truth, semantic snapshots, grant-gated spawn mediation, and
the focusable overlay session lifetime. The Lua
package owns the presentation policy: the `[tools.git]` allowlist, the
working-tree read-scope checks, bounded branch/status/commit listing
composition (128 status entries, 64 commits, a single branch bound of 32),
filtering, and declarative scene composition.

## Failure behavior

The plugin registers statically through lazy triggers and creates no VM until
a command or observation event fires. Missing/denied spawn returns a typed
failure; snapshot refresh is guarded and observation handlers never spawn. A
missing overlay surface fails closed with `E_UI_UNAVAILABLE`, and
`overlay.released` clears the local session so the next open starts clean.

## Security review

- Requests `ui.overlay.focus` (the accepted W-01 coupled grant for a focusable
  overlay and its transient input capture), `terminal.semantic-read` (gates
  `bitty.terminal.snapshot`, the read-only cwd/title observation the panel
  refreshes from), `process.spawn:git`, and a filesystem read-pattern
  declaration. It requests no `panel.provider` or `panel.create`.
- The `[tools.git]` declaration is the accepted Layer 2 slice
  (`bitty` `CTX-0425`): the contract requires `git` to be present and satisfy
  `>=2.30`; this declaration alone does not prove the whole install/activation
  enforcement chain. Raising `required` from `false`
  to `true` is a capability increase whose grant must be re-confirmed.
- Deny by default: any other executable is denied, unknown capability
  identifiers fail validation, and there is no allow-all identifier.

The plugin passes the host-resolved `cwd = "pane"` semantic and never supplies a
path. Core resolves the globally focused view's PTY-child kernel directory at
call time, validates and audits it, and fails closed when it cannot be proved.
That directory is a starting point, not a sandbox: the child keeps the user's OS
authority and repository metadata stays a separate channel. The spawn extra is
explicitly outside stable Plugin API v1, and neither bounds nor compatibility
ranges close the isolation gap.

## Verification plan

Inspect argv restrictions, host grant wiring, pane-cwd resolution, and source
scene composition. Require real-backend repository-isolation tests, a `.git`
metadata policy, and live overlay usability evidence before claiming usability;
mock cwd tests are insufficient.

## Alternatives considered

A bundled-only git-panel was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved it to an independent first-party package with the catalog entry
removed and panel presentation deferred.

## Affected contracts

This update changes no spawn contract, path scope, tool declaration, or panel
grant. See [schemas](schemas.md) for the current integration limits.

## Open points

Validated cwd forwarding, real-path isolation, panel mounting/lifecycle, and
complete install-time tools enforcement remain separate work. There are no
settings reads to declare in this source.

## Acceptance criteria

Repository-aware panel claims require host-validated cwd and independently
verified real backend and presentation behavior.

## P0 Review Sign-off

No new security acceptance is claimed; current isolation is unproven.

## References

- [Git panel plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
