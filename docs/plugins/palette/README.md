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

## Admission criteria

Routes for the inspected focusable-overlay source and its declared setting.

## Authority and status

Registry now pins palette `5169d974` for Core `>=0.0.24` / Plugin API `^1.0`,
with a manifest hash matching the pin. The source acquires, updates, and
releases a focusable overlay with `List`/`Text` content, seeds the filter from
the declared `query` setting, and drains captured keys for live typing. It does
not read `entries`. Registry eligibility is not product verification;
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90)
retains the unaccepted table-shaped entry question.

## Identity

| Field             | Value                                                          |
| ----------------- | -------------------------------------------------------------- |
| Plugin id         | `bitty-terminal.palette`                                       |
| Name              | Palette                                                        |
| Package version   | 0.0.1                                                          |
| Owning repository | <https://github.com/bitty-terminal/palette>                    |
| Lua module        | `lua/palette/`                                                 |
| Capabilities      | `ui.overlay.focus`                                             |
| Lazy commands     | `bitty-terminal.palette:toggle`                                |
| Lazy events       | `focus.changed`, `overlay.released`, `overlay.input.available` |

## Pages

- [Design](design.md) — scope, overlay presentation, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
