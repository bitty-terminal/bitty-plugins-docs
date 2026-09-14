---
title: Statusline plugin documentation
description: Per-plugin index for the statusline composition plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 38
---

# Statusline plugin documentation

Identity, current stage, owning repository, and page links for the independent
first-party statusline package.

## Identity

| Field             | Value                                            |
| ----------------- | ------------------------------------------------ |
| Plugin id         | `bitty-terminal.statusline`                      |
| Name              | Statusline                                       |
| Package version   | 0.1.0                                            |
| Owning repository | <https://github.com/bitty-terminal/statusline>   |
| Lua module        | `lua/statusline/`                                |
| Capabilities      | `terminal.semantic-read`, `ui.rich`              |
| Lazy commands     | none                                             |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed` |

## Stage

Pre-implementation ecosystem: the plugin package, manifest, and presentation
policy are implemented and tested headlessly, while the Bitty host is still
landing the Plugin API v1 statusline bridge. The package is not verified,
compatible, or shipped beyond its manifest `[compat]` ranges.

## Pages

- [Design](design.md) — scope, presentation policy, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft), including the
  statusline and shell-prompt boundary
- [Plugin API v1 Lua Surface RFC](../../../specifications/plugin-api-v1-lua-surface-rfc.md)
