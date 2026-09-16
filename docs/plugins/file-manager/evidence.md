---
title: File manager plugin evidence and links
description: Split decision tasks and validation evidence for the file manager plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 45
---

# File manager plugin evidence and links

## Tasks and provenance

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the file-manager out of the bundled catalog into the
  independent first-party package `bitty-terminal/file-manager`.
- `bitty` `CTX-0399` — owning task for the file-manager split; catalog entry
  removed in `bitty` PR #725 (commit `65aac5c`, 2026-09-15), which removed the
  bundled manifest and `bitty-runtime/src/file_manager.rs`.
- `bitty-plugins` `CTX-0010` — registry entry published as
  `registry/official/file-manager.toml` (`bitty-plugins` PR #23, commit
  `84f0b7d`, 2026-09-15).
- `bitty-plugins-docs` `CTX-0023` — this per-plugin page set; recorded as
  related to the shared review-capture Issue
  [bitty-plugins-docs#43](https://github.com/bitty-terminal/bitty-plugins-docs/issues/43),
  which stays open.
- Revisions inspected as evidence (read-only, 2026-09-16): `file-manager`
  `cd3da87` (2026-09-16); `bitty-plugins` `001ef11`; `bitty` `65aac5c`
  (`--stat` only).
- Scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template).

## Validation

The owning repository
([file-manager](https://github.com/bitty-terminal/file-manager)) provides the
evidence: its `just check` gates, headless manifest and package validation,
LuaLS conformance, and `bitty-plugin-lint` (including the negative-fixture
gate with the byte-identical accepted control). This page set does not restate
that evidence and does not claim independent verification.

## Status

Implemented: package, manifest, and observation-only policy, tested
headlessly; registry entry published. Deferred: panel presentation (pending
the panel-provider contract, `bitty-docs` `CTX-0181`, OQ-058) and `fs.*`
access (no Plugin API v1 `bitty.fs` surface). Verified/shipped: nothing —
the package is not verified, compatible, or shipped beyond the manifest
`[compat]` ranges, and no product release contains it.

## Related

- [File manager plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
