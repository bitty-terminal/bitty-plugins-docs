---
title: Statusline plugin evidence and links
description: Split decision tasks and validation evidence for the statusline plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 41
---

# Statusline plugin evidence and links

## Tasks and provenance

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the statusline out of the bundled catalog into the independent
  first-party package `bitty-terminal/statusline`.
- `bitty` `CTX-0398` — owning task for the statusline split.
- `bitty-plugins-docs` `CTX-0004` — this per-plugin page set; `CTX-0002`
  recorded the statusline split implementation status.
- Scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template).

## Validation

The owning repository
([statusline](https://github.com/bitty-terminal/statusline)) provides the
evidence: its `just check` gates, headless manifest and package validation,
LuaLS conformance, and `bitty-plugin-lint`. This page set does not restate that
evidence and does not claim independent verification.

## Status

Package, manifest, and presentation policy implemented and tested headlessly;
the host statusline bridge is still landing. Not verified, compatible, or
shipped beyond the manifest `[compat]` ranges. The catalog removal is tracked
as open in the roadmap.

## Related

- [Statusline plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
