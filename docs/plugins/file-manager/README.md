---
title: File manager plugin documentation
description: Per-plugin index for the read-only file manager plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 42
---

# File manager plugin documentation

## Admission criteria

Routes for the read-only listing package and deferred panel integration.

## Authority and status

Registry pins file-manager `558bf5a2` for Core `>=0.0.24` / Plugin API `^1.0`,
with a manifest hash matching the pin. It observes terminal cwd/title,
enumerates a caller-chosen root through the host-mediated `bitty.fs.list` under
a read-only `fs.read:**` scope, and presents the listing as a `bottom` band
block. It does not write, mutate, or mount a panel. `root`, `entries`, and
`width` are declared settings; table-shaped `entries` remains unaccepted in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
Registry eligibility establishes no real-path isolation, panel, or release
usability.

## Identity

| Field             | Value                                                             |
| ----------------- | ----------------------------------------------------------------- |
| Plugin id         | `bitty-terminal.file-manager`                                     |
| Name              | File Manager                                                      |
| Package version   | 0.1.0                                                             |
| Owning repository | <https://github.com/bitty-terminal/file-manager>                  |
| Lua module        | `lua/file-manager/`                                               |
| Capabilities      | `terminal.semantic-read`, `ui.rich`, `fs.read:**`                 |
| Lazy commands     | `bitty-terminal.file-manager:open`, `:preview`, `:rename`         |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed` |

## Pages

- [Design](design.md) — scope, read-only policy, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
