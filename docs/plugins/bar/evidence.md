---
title: Bar plugin evidence and links
description: Tasks decisions and validation evidence for the unified bar plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 53
---

# Bar plugin evidence and links

## Purpose

Record the current bar implementation separately from candidate chrome design.

## Source identity

Registry [#102](https://github.com/bitty-terminal/bitty-plugins/pull/102), merge
`b1eade711049e8c852a076fbc9db3700dd70fce9`, pins bar
`18d1a5b6dc86af4a42546f20f1610e69af58071d` (bar main). The manifest and
`lua/bar/init.lua` were independently inspected. Compatibility declarations
are Core `>=0.0.24,<0.1` and Plugin API `^1.0`; package version is `0.0.1`.
Registry [#92](https://github.com/bitty-terminal/bitty-plugins/pull/92),
merge `ba4d9d1268bfd877ab60413f496e95b6cae41168`, supplied the earlier
pin `f007a03d`.

Historical design context:

- [ADR 0014: Workspace as Core Mechanism with Plugin-Only Presentation](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0014-workspace-core-presentation-plugins.md)
  (accepted 2026-10-01) established that workspace lifecycle is Core mechanism
  while presentation (bars, tabs) is strictly plugin-only.
- [Sparse Workspaces and Unified Chrome Bar Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/sparse-workspace-unified-bar-candidate.md)
  specifies Core's sparse on-demand workspace addressing, switch-or-create
  semantics, prune-on-empty cleanup, and the single-row `ChromeInsets`
  reservation.
- `bitty` `CTX-0398` — split statusline into an independent first-party package.
- `bitty` `CTX-0886` — trimmed Core to minimal terminal platform mechanism.
- Scaffolded following
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template)
  and validated against
  [bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk).

## Disposition

Implemented: declared chassis scalars, the `label` capsule-label mode, and
group string arrays; shipped default plan of workspaces left, clock center, and
empty right; `top`/`bottom` mounts under `ui.rich`; decorated `Text` in a `Row`;
workspace focus and optional provider policy. Per-module option keys are empty
string arrays, so map options remain ineffective on the strict host. No new
`Capsule` or widget kind is implemented by the pin. Registry eligibility does
not prove live presentation or release delivery.

## Verification backlog

Planned validation for the owning repository (`bitty-terminal/bar`):

- `just check` quality gates including LuaLS type-checking and `bitty-plugin-lint`.
- Headless mock observation stream tests confirming correct rendering of
  sparse, non-contiguous workspace sequences (`[1, 2, 4, 7, 9]`).
- Pointer tests verifying the actual `bitty-terminal.bar:focus` binding and
  its `workspace.control` denial behavior.
- Verification that `ChromeInsets` ensures zero occlusion of the terminal grid.

Per-module option shape remains
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
True alignment and broader workspace interactions remain candidate. This docs
pass reruns no live UI or plugin product tests.

### References

- [Bar plugin documentation](README.md)
- [Design](design.md)
- [Schemas and contracts](schemas.md)
- [Statusline plugin evidence](../statusline/evidence.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
