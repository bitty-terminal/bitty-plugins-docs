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

## Admission criteria

Routes for observation-only file-manager policy and deferred host integration.

## Authority and status

Registry #92 pins file-manager `bf3a17ea` for Core `>=0.0.24,<0.1` / Plugin
API `^1.0`. It observes terminal cwd/title and shapes supplied paths; it does
not enumerate/read/rename host files or mount a panel. Undeclared settings
use guarded fallback, and table-shaped `entries` remains unaccepted in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
Registry eligibility establishes no filesystem, panel, or release usability.

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

## Pages

- [Design](design.md) — scope, observation-only policy, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
