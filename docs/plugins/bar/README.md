---
title: Bar plugin documentation
description: Per-plugin index for the unified single-row bar presentation plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 50
---

# Bar plugin documentation

Identity, current stage, owning repository, and page links for the candidate
unified bar presentation package.

## Identity

| Field             | Value                                                                                                                                                 |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugin id         | `bitty-terminal.bar`                                                                                                                                  |
| Name              | Unified Bar                                                                                                                                           |
| Package version   | 0.0.1                                                                                                                                                 |
| Owning repository | <https://github.com/bitty-terminal/bar>                                                                                                               |
| Lua module        | `lua/bar/`                                                                                                                                            |
| Capabilities      | `ui.chrome-band`, `terminal.semantic-read`                                                                                                            |
| Lazy commands     | none                                                                                                                                                  |
| Lazy events       | `workspace.snapshot`, `workspace.created`, `workspace.destroyed`, `focus.changed`, `terminal.cwd-changed`, `terminal.title-changed`, `runtime.status` |

## Stage

Candidate specification: consolidates the separate tab strip and statusline
surfaces into a single 1-row edge-band presentation plugin, conforming to the
[ADR 0014](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0014-workspace-core-presentation-plugins.md)
mechanism and presentation boundary and the candidate
[Sparse Workspaces and Unified Chrome Bar Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/sparse-workspace-unified-bar-candidate.md).
The package is not verified, compatible, or shipped.

## Pages

- [Design](design.md) — scope, Waybar-style composition, pointer interactions, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest fields, configuration keys, observation events, and commands.
- [Evidence and links](evidence.md) — ADR-0014 alignment, candidate Core contracts, and validation evidence.

## Related

- [Plugin documentation](../README.md)
- [Statusline plugin documentation](../statusline/README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
- [Sparse Workspaces and Unified Chrome Bar Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/sparse-workspace-unified-bar-candidate.md)
