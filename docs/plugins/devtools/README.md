---
title: DevTools plugin documentation
description: Per-plugin index for the read-only runtime inspection and event tracing plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 58
---

# DevTools plugin documentation

## Admission criteria

Routes for the read-only DevTools manifest, its declared configuration surface,
and the candidate GUI direction that this package does not implement.

## Authority and status

Registry pins DevTools `4e8e9680` for Core `>=0.0.21` / Plugin API `^1.0`, with a
manifest hash matching the pin. The source is command and notification only: it
reads `bitty.debug.inspect`/`trace`/`trace_get` under `debug.inspect` and
`debug.trace` and reports bounded text summaries through `platform.notify`. It
declares no `[settings]` and reads no plugin settings, so there is no
configuration gap to reconcile. `debug.control` and every `ui.*` identifier are
absent by design. Registry eligibility is not usability or release
verification, and the candidate GUI live view remains unimplemented.

## Identity

| Field             | Value                                                                                |
| ----------------- | ------------------------------------------------------------------------------------ |
| Plugin id         | `bitty-featured.devtools`                                                            |
| Name              | Bitty DevTools                                                                       |
| Package version   | 0.1.0                                                                                |
| Owning repository | <https://github.com/bitty-terminal/devtools>                                         |
| Lua module        | `lua/devtools/`                                                                      |
| Capabilities      | `debug.inspect`, `debug.trace`, `platform.notify`                                    |
| Lazy commands     | `plugins`, `commands`, `events`, `grants`, `trace-start`, `trace-stop`, `trace-dump` |
| Lazy events       | Nine v1 observation kinds; see [schemas](schemas.md)                                 |

## Pages

- [Schemas and contracts](schemas.md) — manifest surface, trace event set, and
  the absent settings declaration.

## Related

- [Plugin documentation](../README.md)
- [DevTools GUI live-view candidate](../../../extensibility/devtools-gui-live-view-candidate.md)
  (draft candidate; no interface accepted)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
