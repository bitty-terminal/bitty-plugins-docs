---
title: Palette plugin evidence and links
description: Split decision tasks and validation evidence for the palette plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 37
---

# Palette plugin evidence and links

## Purpose

Record the focusable overlay source and the unresolved settings population gap.

## Source identity

Registry [#92](https://github.com/bitty-terminal/bitty-plugins/pull/92), merge
`ba4d9d1268bfd877ab60413f496e95b6cae41168`, pins palette
`c17dc321c8c7d4c8a1b8c5e7853d33e943af7c3d`. Manifest, entry point, filter,
and scene were independently inspected. Package version is `0.0.1`, with
Core `>=0.0.24,<0.1` and Plugin API `^1.0` declarations.

Historical context:

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the palette out of the bundled catalog into the independent
  first-party package `bitty-terminal/palette`.
- `bitty` `CTX-0397` — owning task for the palette split.
- `bitty-plugins-docs` `CTX-0004` — this per-plugin page set; `CTX-0002`
  recorded the palette split implementation status.
- Scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template).

## Disposition

The pin uses `ui.overlay.focus` and acquire/update/release of a session-owned
surface, with `List`/`Text` content. Guarded `entries`/`query` reads are
undeclared and use empty defaults. It does not implement live query typing.
Compatibility declarations and registry inclusion do not prove an interactive
or populated palette on the strict host.

## Verification backlog

The owning repository ([palette](https://github.com/bitty-terminal/palette))
provides the evidence: its `just check` gates, headless manifest and package
validation, LuaLS conformance, and `bitty-plugin-lint`. This page set does not
restate that evidence and does not claim independent verification.

Declarations and the unaccepted table-shaped `entries` question remain
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
This source inspection does not rerun plugin or live overlay tests.

### References

- [Palette plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
