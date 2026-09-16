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

Identity, current stage, owning repository, and page links for the independent
first-party git-panel package.

## Identity

| Field             | Value                                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------- |
| Plugin id         | `bitty-terminal.git-panel`                                                                               |
| Name              | Git Panel                                                                                                |
| Package version   | 0.1.0                                                                                                    |
| Owning repository | <https://github.com/bitty-terminal/git-panel>                                                            |
| Lua module        | `lua/git-panel/`                                                                                         |
| Capabilities      | `panel.provider`, `panel.create`, `terminal.semantic-read`, `process.spawn:git`, `fs.read:~/projects/**` |
| System tool       | `[tools.git]` (`required = true`, `version = ">=2.30"`)                                                  |
| Lazy commands     | `bitty-terminal.git-panel:open`, `:status`, `:diff`, `:log`, `:branch`                                   |
| Lazy events       | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed`                                        |

## Stage

Pre-implementation ecosystem: the plugin package, manifest, and policy are
implemented and tested headlessly, while the Bitty host is still landing the
Layer 2 spawn bridge and the panel-provider contract. The package is not
verified, compatible, or shipped beyond its manifest `[compat]` ranges.

## Pages

- [Design](design.md) — scope, tiled presentation, and capability boundaries.
- [Schemas and contracts](schemas.md) — manifest surface, `[tools.git]`, and version ranges.
- [Evidence and links](evidence.md) — split decision, tasks, and validation.

## Related

- [Plugin documentation](../README.md)
- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Reuse and Provider Ecology RFC](../../../specifications/plugin-reuse-and-providers.md)
