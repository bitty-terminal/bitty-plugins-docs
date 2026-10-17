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

## Purpose

Record observation-only implementation evidence without implying filesystem or
panel usability.

## Source identity

Registry [#92](https://github.com/bitty-terminal/bitty-plugins/pull/92), merge
`ba4d9d1268bfd877ab60413f496e95b6cae41168`, pins file-manager
`bf3a17ea6aa1ea0fcc3a45a13ceaa069e83b71d4`. Manifest and Lua were inspected
independently. Package version is `0.1.0`, with Core `>=0.0.24,<0.1` and
Plugin API `^1.0` declarations.

Historical context:

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

## Disposition

Implemented: bounded path/listing/preview/rename-pair composition with terminal
observation only. The pin has no filesystem or panel calls and requests only
`terminal.semantic-read`. Guarded undeclared `root`/`entries` reads fall back
to explicit arguments, cached cwd, or an empty listing. It cannot enumerate,
read, or rename real files or present a panel. Registry eligibility is not a
product usability or release claim.

## Verification backlog

The owning repository
([file-manager](https://github.com/bitty-terminal/file-manager)) provides the
evidence: its `just check` gates, headless manifest and package validation,
LuaLS conformance, and `bitty-plugin-lint` (including the negative-fixture
gate with the byte-identical accepted control). This page set does not restate
that evidence and does not claim independent verification.

Remaining work includes
[settings issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90),
an accepted entry shape, production filesystem authority/backend, and panel
lifecycle. Source inspection and pure path tests cannot establish host
real-path isolation. This docs pass runs no live filesystem campaign.

### References

- [File manager plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
