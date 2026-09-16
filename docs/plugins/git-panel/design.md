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

## Problem

Users need git branch, status, diff, and log presentation inside the terminal
workspace without shelling out through unconstrained process authority.

## Scope

In scope: a tiled Panel git branch/status/diff/log presentation over
allowlisted `process.spawn:git` with the manifest-declared `[tools.git]`
contract. The Lua policy is pure plus Layer 2 system-CLI reuse: the allowlist
permits only the `git` binary with the seven read-only verbs (`status`,
`diff`, `log`, `branch`, `show`, `rev-parse`, `ls-files`) under fail-closed
bounds (at most 32 args of at most 256 bytes each), and the working-tree read
scope stays at `~/projects/**`.

Out of scope: terminal truth, input, parser, and render hot paths; arbitrary
process spawn, shell interpolation, network, clipboard, terminal-input, or
persistent-state authority; and write verbs beyond the allowlisted read-only
set. Panel presentation stays host-owned: the package composes declarative
`List`/`Text` panel content, but it cannot present a panel until the
panel-provider contract is accepted. That deferred mounting is an explicit
non-goal of the current package, not implied behavior.

## Mechanism and policy split

Core (host) owns the Panel Runtime, terminal truth, semantic snapshots, and —
once the deferred contracts land — panel mounting and spawn mediation. The Lua
package owns the presentation policy: the `[tools.git]` allowlist, the
working-tree read-scope checks, bounded branch/status/commit listing
composition (128 status entries, 64 commits, a single branch bound of 32),
filtering, and declarative scene composition.

## Capability and trust boundaries

- Requests `panel.provider` and `panel.create` (back the tiled Panel Runtime
  registration), `terminal.semantic-read` (gates `bitty.terminal.snapshot`,
  the read-only cwd/title observation the panel refreshes from),
  `process.spawn:git` (the closed `process.spawn` family plus the `:git`
  parameter — only the `git` binary spawns), and `fs.read:~/projects/**`
  (the working-tree read scope; symlinks and devices rejected per host
  policy). All identifiers are unchanged from the former bundled manifest.
- The `[tools.git]` declaration is the accepted Layer 2 slice
  (`bitty` `CTX-0425`): `git` must be present and satisfy `>=2.30`, otherwise
  activation fails closed with a diagnostic. Raising `required` from `false`
  to `true` is a capability increase whose grant must be re-confirmed.
- Deny by default: any other executable is denied, unknown capability
  identifiers fail validation, and there is no allow-all identifier.

## Failure behavior

The plugin registers statically through lazy triggers and creates no VM until
a command or observation event fires. Missing `git`, an unsatisfied version
constraint, or a denied snapshot degrades to no git-panel surface rather than
a broken panel; observation handlers refresh cached state and never spawn.

## Alternatives considered

A bundled-only git-panel was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved it to an independent first-party package with the catalog entry
removed and panel presentation deferred.

## Related

- [Git panel plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
