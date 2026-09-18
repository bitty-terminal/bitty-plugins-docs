---
title: File manager plugin documentation
description: Per-plugin index for the observation-only file manager plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 42
---

# File manager plugin documentation

Identity, current stage, owning repository, and page links for the independent
first-party file-manager package.

## Identity

| Field             | Value                                                             |
| ----------------- | ----------------------------------------------------------------- |
| Plugin id         | `bitty-terminal.file-manager`                                     |
| Name              | File Manager                                                      |
| Package version   | 0.1.0                                                             |
| Owning repository | <https://github.com/bitty-terminal/file-manager>                  |
| Lua module        | `lua/file-manager/`                                               |
| Capabilities      | `terminal.semantic-read`                                          |
| Lazy commands     | `bitty-terminal.file-manager:open`, `:preview`, `:rename`         |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed` |

## Stage

Pre-implementation ecosystem: the plugin package, manifest, and policy are
implemented and tested headlessly, while the Bitty host is still landing the
host-mediated filesystem bridge. Panel presentation stays deferred pending the
panel-provider contract. The package is not verified, compatible, or shipped
beyond its manifest `[compat]` ranges.

## Pages

- [Design](design.md) — scope, observation-only policy, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
