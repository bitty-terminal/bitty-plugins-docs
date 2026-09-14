---
title: Palette plugin documentation
description: Per-plugin index for the command palette and picker plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 34
---

# Palette plugin documentation

Identity, current stage, owning repository, and page links for the independent
first-party palette package.

## Identity

| Field             | Value                                       |
| ----------------- | ------------------------------------------- |
| Plugin id         | `bitty-terminal.palette`                    |
| Name              | Palette                                     |
| Package version   | 0.1.0                                       |
| Owning repository | <https://github.com/bitty-terminal/palette> |
| Lua module        | `lua/palette/`                              |
| Capabilities      | `ui.rich`, `ui.overlay`                     |
| Lazy commands     | `bitty-terminal.palette:toggle`             |
| Lazy events       | `focus.changed`                             |

## Stage

Pre-implementation ecosystem: the plugin package, manifest, and policy are
implemented and tested headlessly, while the Bitty host is still landing the
Plugin API v1 overlay bridge. The package is not verified, compatible, or
shipped beyond its manifest `[compat]` ranges.

## Pages

- [Design](design.md) — scope, overlay presentation, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin API v1 Lua Surface RFC](../../../specifications/plugin-api-v1-lua-surface-rfc.md)
