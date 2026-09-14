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

## Tasks and provenance

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the palette out of the bundled catalog into the independent
  first-party package `bitty-terminal/palette`.
- `bitty` `CTX-0397` — owning task for the palette split.
- `bitty-plugins-docs` `CTX-0004` — this per-plugin page set; `CTX-0002`
  recorded the palette split implementation status.
- Scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template).

## Validation

The owning repository ([palette](https://github.com/bitty-terminal/palette))
provides the evidence: its `just check` gates, headless manifest and package
validation, LuaLS conformance, and `bitty-plugin-lint`. This page set does not
restate that evidence and does not claim independent verification.

## Status

Package, manifest, and policy implemented and tested headlessly; the host
overlay bridge is still landing. Not verified, compatible, or shipped beyond
the manifest `[compat]` ranges.

## Related

- [Palette plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
