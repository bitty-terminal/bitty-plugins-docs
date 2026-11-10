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

## Purpose

Record the current source boundary and remaining verification for activity.

## Source identity

Registry pins activity `0ea39e13`. Its manifest and Lua were independently
inspected for this synchronization. The manifest version is `0.0.1`, Core range
`>=0.0.24`, and Plugin API range `^1.0`.

Historical context:

- `bitty` `CTX-0221` — first featured-plugin-wave pathfinder; scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template)
  and validated against
  [bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk).
- `bitty-plugins-docs` `CTX-0004` — this per-plugin page set.

## Disposition

Implementation is present and the registry pin is eligible. The manifest
declares both `retention_days` (int, `7`) and `store_command_args` (bool,
`false`) and the guarded reads use the declared values or their local defaults.
Enabling the latter never stores arguments in v1.
Registry inclusion is not a claim of release delivery or end-to-end usability.

## Verification backlog

The owning repository ([activity](https://github.com/bitty-terminal/activity))
provides the evidence: its `just check` gates, behaviour tests under `tests/`,
LuaLS conformance, manifest validation, and `bitty-plugin-lint`. This page set
does not restate that evidence and does not claim independent verification.

Remaining declaration work is
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
This pass verifies source facts, not a live plugin or notification campaign.

### References

- [Activity plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
