---
title: Activity plugin documentation
description: Per-plugin index for the privacy-first activity timeline plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 30
---

# Activity plugin documentation

Identity, current stage, owning repository, and page links for the first
featured-wave official plugin.

## Identity

| Field             | Value                                                              |
| ----------------- | ------------------------------------------------------------------ |
| Plugin id         | `bitty-featured.activity`                                          |
| Name              | Bitty Activity                                                     |
| Package version   | 0.1.0                                                              |
| Owning repository | <https://github.com/bitty-terminal/activity>                       |
| Lua module        | `lua/activity/`                                                    |
| Capabilities      | `terminal.semantic-read`, `platform.notify`                        |
| Lazy commands     | `bitty-featured.activity:summary`, `bitty-featured.activity:clear` |

## Stage

Implementation present, host integration still landing. The owning repository
implements the v1 privacy-first timeline against the accepted Plugin API v1
surface and validates it with its own gates, behaviour tests, LuaLS
conformance, and `bitty-plugin-lint`. It is not verified, compatible, or
shipped, and no product release contains it.

## Pages

- [Design](design.md) — scope, capability and privacy boundaries, failure behavior.
- [Schemas and contracts](schemas.md) — manifest surface and version ranges.
- [Evidence and links](evidence.md) — tasks, validation, and provenance.

## Related

- [Plugin documentation](../README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
- [Plugin Host Runtime RFC](../../../runtime/plugin-host-runtime-rfc.md)
