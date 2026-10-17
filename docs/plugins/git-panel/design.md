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

Current source at registry-pinned `862df1f9`. Core `662212c5` provides the
grant-gated git spawn extra, but repository isolation and panel usability
remain unproven.

## Purpose and scope

Users need git branch, status, diff, and log presentation inside the terminal
workspace without shelling out through unconstrained process authority.

## Normative sources this specification must not weaken

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and isolation. The accepted
[`tools.git` slice](../../../packaging/plugin-reuse-and-providers.md#accepted-toolsgit-contract-v1)
does not create a general stable process API.

## Terminology

Command data and declarative scenes are not a mounted panel. An argv
allowlist constrains verbs; it does not prove which repository executes them.

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
set. Panel presentation stays host-owned: the package composes declarative
`List`/`Text` panel content, but it cannot present a panel until the
panel-provider contract is accepted. That deferred mounting is an explicit
non-goal of the current package, not implied behavior.

## Mechanism and policy split

Core owns terminal truth, semantic snapshots, and grant-gated spawn mediation.
Panel mounting remains deferred. The Lua
package owns the presentation policy: the `[tools.git]` allowlist, the
working-tree read-scope checks, bounded branch/status/commit listing
composition (128 status entries, 64 commits, a single branch bound of 32),
filtering, and declarative scene composition.

## Failure behavior

The plugin registers statically through lazy triggers and creates no VM until
a command or observation event fires. Missing/denied spawn returns a typed
failure; snapshot refresh is guarded and observation handlers never spawn.
No mounted git-panel surface is implemented by this source.

## Security review

- Requests `panel.provider` and `panel.create` (deferred candidates pending
  the panel-provider contract, `bitty-docs` `CTX-0181`, OQ-058, backing the
  tiled Panel Runtime registration), `terminal.semantic-read` (gates `bitty.terminal.snapshot`,
  the read-only cwd/title observation the panel refreshes from),
  `process.spawn:git`, and a filesystem read-pattern declaration. The Lua
  lexical path filter does not establish production real-path, symlink, or
  device enforcement.
- The `[tools.git]` declaration is the accepted Layer 2 slice
  (`bitty` `CTX-0425`): the contract requires `git` to be present and satisfy
  `>=2.30`; this declaration alone does not prove the whole install/activation
  enforcement chain. Raising `required` from `false`
  to `true` is a capability increase whose grant must be re-confirmed.
- Deny by default: any other executable is denied, unknown capability
  identifiers fail validation, and there is no allow-all identifier.

The plugin sends `{ cwd = cache.cwd }` as the second spawn argument. Core
`662212c5` reads only argv and does not forward that option. A real backend
exists, but it does not prove the pane-repository boundary. The spawn extra is
explicitly outside stable Plugin API v1. Neither bounds nor compatibility
ranges close that isolation gap.

## Verification plan

Inspect argv restrictions, host grant wiring, source scene composition, and
the missing cwd forwarding. Require actual backend repository-isolation and
accepted provider-lifecycle tests before claiming usability; mock cwd tests
are insufficient.

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
