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

## Purpose

Record the difference between bounded git command policy, a real Core spawn
extra, and unproven repository/panel usability.

## Source identity

Registry [#92](https://github.com/bitty-terminal/bitty-plugins/pull/92), merge
`ba4d9d1268bfd877ab60413f496e95b6cae41168`, pins git-panel
`862df1f90ad137eb1e92379ac0710c6cd22c7f21`. The manifest, entry point, scope,
and scene were independently inspected against Core `662212c5`'s
`crates/bitty-lua/src/host.rs` spawn callback. Package version is `0.1.0`,
with Core `>=0.0.24,<0.1` and Plugin API `^1.0` declarations.

Historical context:

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the git-panel out of the bundled catalog into the
  independent first-party package `bitty-terminal/git-panel`.
- `bitty` `CTX-0400` — owning task for the git-panel split; catalog entry
  removed in `bitty` PR #713 (commit `e84da34`, 2026-09-15), which removed the
  bundled manifest and the `bitty-runtime::git_panel` review implementation.
- `bitty` `CTX-0425` — accepted the Layer 2 `[tools.git]` declaration (v1),
  recorded canonically in the
  [Plugin Reuse and Provider Ecology RFC](../../../packaging/plugin-reuse-and-providers.md#accepted-toolsgit-contract-v1).
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

## Disposition

The grant-gated Core git backend exists, but its argv-only Lua bridge ignores
the plugin's second cwd argument. Intended repository isolation is unproven.
The plugin returns data and composes scenes without mounting a panel, and
does not call filesystem APIs. It has no settings reads or declaration gap.
Registry eligibility does not prove product usability or release delivery.

## Verification backlog

The owning repository
([git-panel](https://github.com/bitty-terminal/git-panel)) provides the
evidence: its `just check` gates, headless manifest and package validation,
LuaLS conformance, and `bitty-plugin-lint` (including the negative-fixture
gate with the accepted control). This page set does not restate that evidence
and does not claim independent verification.

Require accepted/validated cwd forwarding and real-backend isolation tests,
an accepted panel-provider lifecycle, and complete install-time tools/grant
evidence. Dispatch authorizer tests do not prove those paths. This pass
independently checks source only and runs no live git or panel campaign.

### References

- [Git panel plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
