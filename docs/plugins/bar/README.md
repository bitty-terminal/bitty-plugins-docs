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

## Admission criteria

Routes for the current bar source and explicitly deferred chrome direction.

## Authority and status

Registry #102 pins bar `18d1a5b6` for Core `>=0.0.24,<0.1` / Plugin API `^1.0`.
The source mounts a text row in `top` or `bottom`, with declared chassis and
module-list settings. A fresh install paints workspaces on the left and the
clock in the center group, with an empty right group; the `label` setting
selects index, name, or both. Module option maps remain ineffective under the
current strict kinds and stay with
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
True alignment and richer widgets remain candidate. Settings are read at
activation and pinned to the generation; `config.reloaded` does not hot-reload
them. Source presence and registry eligibility establish no live presentation
or release verification.

## Identity

| Field             | Value                                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------ |
| Plugin id         | `bitty-terminal.bar`                                                                                               |
| Name              | Unified Bar                                                                                                        |
| Package version   | 0.0.1                                                                                                              |
| Owning repository | <https://github.com/bitty-terminal/bar>                                                                            |
| Lua module        | `lua/bar/`                                                                                                         |
| Capabilities      | `ui.rich`, workspace read/control, notifications, and separately granted provider heads; see [schemas](schemas.md) |
| Lazy commands     | Focus, detail, and provider controls; see [schemas](schemas.md#events-and-commands)                                |
| Lazy events       | Workspace changes and `config.reloaded`; see [schemas](schemas.md#events-and-commands)                             |

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
