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

Registry pins palette `5169d974`. Manifest, entry point, filter, and scene were
independently inspected. Package version is `0.0.1`, with Core `>=0.0.24` and
Plugin API `^1.0` declarations.

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
surface, with `List`/`Text` content. `query` is declared and seeds the filter;
`entries` is not read. Compatibility declarations and registry inclusion do not
prove an interactive palette on a given host build.

## Verification backlog

The owning repository ([palette](https://github.com/bitty-terminal/palette))
provides the evidence: its `just check` gates, headless manifest and package
validation, LuaLS conformance, and `bitty-plugin-lint`. This page set does not
restate that evidence and does not claim independent verification.

The unaccepted table-shaped `entries` question remains
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
This source inspection does not rerun plugin or live overlay tests.

### References

- [Palette plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
