---
title: Statusline plugin documentation
description: Per-plugin index for the archived statusline composition plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 38
---

# Statusline plugin documentation

## Admission criteria

Routes for the archived statusline source, its undeclared settings, and the
successor presentation surface.

## Authority and status

The `bitty-terminal/statusline` repository is archived on GitHub (last push
2026-10-02) and has no entry under `registry/official/` in `bitty-plugins`. Its
archived manifest declares `>=0.1,<1.0`, so Core 0.0.24 eligibility must not be
inferred from it. Six guarded formatting-setting reads lack declarations and use
local defaults on the strict host; with the repository archived, no active owner
can declare them. The bar manifest describes itself as carrying consolidated
workspace-bar, tabs, and statusline presentation, which is a separate contract
from this archived source. Current `Row`/`Text` composition is not end-to-end or
release verification.

## Identity

| Field             | Value                                                                                |
| ----------------- | ------------------------------------------------------------------------------------ |
| Plugin id         | `bitty-terminal.statusline`                                                          |
| Name              | Statusline                                                                           |
| Package version   | 0.0.1                                                                                |
| Owning repository | <https://github.com/bitty-terminal/statusline> (archived)                            |
| Lua module        | `lua/statusline/`                                                                    |
| Capabilities      | `terminal.semantic-read`, `ui.rich`                                                  |
| Lazy commands     | none                                                                                 |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`, `terminal.opened` |

## Pages

- [Design](design.md) — scope, presentation policy, and capability boundaries.
- [Schemas and contracts](schemas.md) — archived manifest surface, version
  ranges, and the settings gap.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bar plugin documentation](../bar/README.md) — consolidated bar and statusline
  presentation surface
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft), including the
  statusline and shell-prompt boundary
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
