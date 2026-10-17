---
title: Git panel plugin documentation
description: Per-plugin index for the allowlisted CLI tiled git panel plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 46
---

# Git panel plugin documentation

## Admission criteria

Routes for bounded git command policy and unresolved repository/panel integration.

## Authority and status

Registry #92 pins git-panel `862df1f9` for Core `>=0.0.24,<0.1` / Plugin API
`^1.0`. Core `662212c5` has a real grant-gated git spawn extra, outside stable
v1, but its Lua bridge ignores the plugin's requested cwd option. Repository
isolation is unproven. The source returns data and builds scenes without
mounting a panel; it has no settings reads. Eligibility establishes no safe
repository-aware panel or release verification.

## Identity

| Field             | Value                                                                                                          |
| ----------------- | -------------------------------------------------------------------------------------------------------------- |
| Plugin id         | `bitty-terminal.git-panel`                                                                                     |
| Name              | Git Panel                                                                                                      |
| Package version   | 0.1.0                                                                                                          |
| Owning repository | <https://github.com/bitty-terminal/git-panel>                                                                  |
| Lua module        | `lua/git-panel/`                                                                                               |
| Capabilities      | `panel.provider`, `panel.create`, `terminal.semantic-read`, `process.spawn:git`, and a filesystem read pattern |
| System tool       | `[tools.git]` (`required = true`, `version = ">=2.30"`)                                                        |
| Lazy commands     | `bitty-terminal.git-panel:open`, `:status`, `:diff`, `:log`, `:branch`                                         |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`                                              |

## Pages

- [Design](design.md) — scope, tiled presentation, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface, `[tools.git]`, and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Reuse and Provider Ecology RFC](../../../packaging/plugin-reuse-and-providers.md)
