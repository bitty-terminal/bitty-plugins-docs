---
title: Activity plugin evidence and links
description: Tasks validation and provenance evidence for the activity timeline plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 33
---

# Activity plugin evidence and links

## Tasks and provenance

- `bitty` `CTX-0221` — first featured-plugin-wave pathfinder, planning note
  `PX-1199`; scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template)
  and validated against
  [bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk).
- `bitty-plugins-docs` `CTX-0004` — this per-plugin page set.

## Validation

The owning repository ([activity](https://github.com/bitty-terminal/activity))
provides the evidence: its `just check` gates, behaviour tests under `tests/`,
LuaLS conformance, manifest validation, and `bitty-plugin-lint`. This page set
does not restate that evidence and does not claim independent verification.

## Status

Implemented in the plugin repository; not verified, compatible, or shipped.
No product release contains the plugin. Absence of further evidence is
recorded rather than inferred.

## Related

- [Activity plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
