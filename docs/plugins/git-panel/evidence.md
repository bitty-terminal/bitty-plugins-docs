---
title: Git panel plugin evidence and links
description: Split decision tasks and validation evidence for the git panel plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 49
---

# Git panel plugin evidence and links

## Tasks and provenance

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the git-panel out of the bundled catalog into the
  independent first-party package `bitty-terminal/git-panel`.
- `bitty` `CTX-0400` — owning task for the git-panel split; catalog entry
  removed in `bitty` PR #713 (commit `e84da34`, 2026-09-15), which removed the
  bundled manifest and the `bitty-runtime::git_panel` review implementation.
- `bitty` `CTX-0425` — accepted the Layer 2 `[tools.git]` declaration (v1),
  recorded canonically in the
  [Plugin Reuse and Provider Ecology RFC](../../../specifications/plugin-reuse-and-providers.md#accepted-toolsgit-contract-v1).
- `bitty-plugins` `CTX-0008` — registry entry published as
  `registry/official/git-panel.toml` (`bitty-plugins` PR #19, commit
  `9899c1e`, 2026-09-15).
- `bitty-plugins-docs` `CTX-0023` — this per-plugin page set; recorded as
  related to the shared review-capture Issue
  [bitty-plugins-docs#43](https://github.com/bitty-terminal/bitty-plugins-docs/issues/43),
  which stays open.
- Revisions inspected as evidence (read-only, 2026-09-16): `git-panel`
  `528a62b` (2026-09-16); `bitty-plugins` `001ef11`; `bitty` `e84da34`
  (`--stat` only).
- Scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template).

## Validation

The owning repository
([git-panel](https://github.com/bitty-terminal/git-panel)) provides the
evidence: its `just check` gates, headless manifest and package validation,
LuaLS conformance, and `bitty-plugin-lint` (including the negative-fixture
gate with the accepted control). This page set does not restate that evidence
and does not claim independent verification.

## Status

Implemented: package, manifest, allowlist policy, and `[tools.git]`
declaration, tested headlessly; registry entry published. Deferred: panel
presentation (pending the panel-provider contract, `bitty-docs` `CTX-0181`,
OQ-058) and host-side `[tools.*]` table enforcement in the install path
(follow-up work). Verified/shipped: nothing — the package is not verified,
compatible, or shipped beyond the manifest `[compat]` ranges, and no product
release contains it.

## Related

- [Git panel plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
