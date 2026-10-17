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

## Admission criteria

Routes for statusline source and pending settings/eligibility work.

## Authority and status

Source `2b9973c7` is not one of registry #92's five eligible pins. Its manifest
range remains `>=0.1,<1.0`; do not infer Core 0.0.24 eligibility. Six guarded
formatting-setting reads lack declarations and use local defaults on the strict
host. [Issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90)
retains that work. Current `Row`/`Text` composition is not end-to-end or release
verification.

## Identity

| Field             | Value                                                                                |
| ----------------- | ------------------------------------------------------------------------------------ |
| Plugin id         | `bitty-terminal.statusline`                                                          |
| Name              | Statusline                                                                           |
| Package version   | 0.0.1                                                                                |
| Owning repository | <https://github.com/bitty-terminal/statusline>                                       |
| Lua module        | `lua/statusline/`                                                                    |
| Capabilities      | `terminal.semantic-read`, `ui.rich`                                                  |
| Lazy commands     | none                                                                                 |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`, `terminal.opened` |

## Pages

- [Design](design.md) — scope, presentation policy, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft), including the
  statusline and shell-prompt boundary
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
