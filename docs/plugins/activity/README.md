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

## Admission criteria

Routes for the independently inspected activity manifest and implementation.

## Authority and status

Registry pins activity `0ea39e13` for Core `>=0.0.24` / Plugin API `^1.0`, with a
manifest hash matching the pin. Implementation is present; both settings keys it
reads (`retention_days`, `store_command_args`) are declared, and command
arguments are never stored. This source snapshot establishes no end-to-end or
release verification.

## Identity

| Field             | Value                                                              |
| ----------------- | ------------------------------------------------------------------ |
| Plugin id         | `bitty-featured.activity`                                          |
| Name              | Bitty Activity                                                     |
| Package version   | 0.0.1                                                              |
| Owning repository | <https://github.com/bitty-terminal/activity>                       |
| Lua module        | `lua/activity/`                                                    |
| Capabilities      | `terminal.semantic-read`, `platform.notify`                        |
| Lazy commands     | `bitty-featured.activity:summary`, `bitty-featured.activity:clear` |

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
