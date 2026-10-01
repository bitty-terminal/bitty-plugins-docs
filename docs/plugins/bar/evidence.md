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

## Tasks and provenance

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

## Validation

Planned validation for the owning repository (`bitty-terminal/bar`):

- `just check` quality gates including LuaLS type-checking and `bitty-plugin-lint`.
- Headless mock observation stream tests confirming correct rendering of
  sparse, non-contiguous workspace sequences (`[1, 2, 4, 7, 9]`).
- Pointer hit-test unit tests verifying that mouse clicks on pills correctly
  dispatch `workspace_focus:N` and `workspace_close_request:N`.
- Verification that `ChromeInsets` ensures zero occlusion of the terminal grid.

## Status

Candidate specification: unifies the separate tab strip and statusline concepts
into a single 1-row edge-band presentation plugin. Not verified, compatible, or
shipped.

## Related

- [Bar plugin documentation](README.md)
- [Design](design.md)
- [Schemas and contracts](schemas.md)
- [Statusline plugin evidence](../statusline/evidence.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
