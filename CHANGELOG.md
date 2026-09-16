# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Repository bootstrap (CTX-0187 Phase 1): docs-quality toolchain (justfile,
  `.github/scripts/check-docs.mjs`, docs-quality workflow), CarryCtx baseline,
  and an empty documentation skeleton.
- Plugin-ecosystem corpus migrated from `bitty-docs` at `c664214` (CTX-0001,
  parent bitty-docs CTX-0187 Phase 3) with path-limited Git history: nine
  specifications, the plugin roadmap, two extensibility contracts, and the
  per-plugin index and template, with cross-repository links rewritten.
- Record plugin-ecosystem direction from the workspace `research` records
  `origin/040.md` and `origin/039.md` as the draft
  `specifications/plugin-ecosystem-model.md`: plugin taxonomy, platform versus
  extension plugins, extension points, manifest candidates, capability
  non-escalation, extension-platform API versioning, the plugin graph, and
  Panel/Activity implications. Research-derived design input, not an accepted
  contract.
- Record the plugin-relevant conclusions of the workspace `research` record
  `origin/041.md` as the draft `specifications/plugin-ipc-boundary.md`:
  out-of-process plugins as a second extension boundary, the plugin event bus,
  the unified Lua/IPC/CLI capability model, capability-token and
  permission-display candidates, crash isolation and supervision questions,
  control-CLI and multi-instance addressing sketches, and the candidate
  three-layer extension framing. Research-derived design input, not an accepted
  contract; the diverging `[permissions]` sketch is marked unaccepted.

### Changed

- Repository metadata baseline: [CONTRIBUTING.md](CONTRIBUTING.md) documents the
  delivery lifecycle, the contributor-branch convention
  (`ctx-XXXX/<type>-<slug>`; external contributors use
  `<handle>/<type>-<slug>`), and the local quality gates;
  [SECURITY.md](SECURITY.md) follows the canonical reporting structure; and
  `.gitattributes` normalizes text files to LF.
