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
- Add `docs/plugins/file-manager/` and `docs/plugins/git-panel/` per-plugin
  page sets (CTX-0023, related to Issue #43): README index, design, schemas,
  and evidence for the independent `bitty-terminal.file-manager`
  (observation-only, `terminal.semantic-read`) and `bitty-terminal.git-panel`
  (tiled panel over allowlisted `process.spawn:git` plus accepted
  `[tools.git]` v1) packages; panel presentation deferred pending the
  panel-provider contract. Nothing here is verified or shipped.
- Record the plugin-side conclusions of the workspace `research` records
  `origin/032.md` (pluggable model access) and `origin/038.md` (pluggable pane
  history) as candidate directions in
  `specifications/plugin-reuse-and-providers.md` with open-item acceptance
  rows: core keeps the provider contract/registry/routing while vendor
  auth/billing/discovery and the management UI live in plugins with opaque
  credential handles composed with the Secrets direction, and durable history
  becomes an official plugin with append-only compressed segments, storage
  capabilities instead of direct database access, command/output/replay tiers,
  and external history tools as providers and sinks. Research-derived design
  input, not an accepted contract; Plugin API v1 defines no `fs.*` Lua entry
  point.
- Disambiguate the `composer` term in the Plugin API v1 Lua Surface RFC and
  record the research 002 API stability priority order as a non-normative
  candidate in the plugin system extension-levels section (CTX-0025, related
  to Issue #43): the user-facing Command Composer stays a terminal-interaction
  concept owned by `bitty-terminal-docs`, distinct from the internal scene-diff
  step. No normative behavior changes.
- Bridge the research-recorded ActivityStack direction to the P2 panel-identity
  candidate (CTX-0026, related to Issue #43): the UI extensibility P2 section
  now points plugin authors to the Plugin Ecosystem Model section 9.2 for the
  Panel-is-not-Activity and session-survival direction (acceptance path: the
  Panel Runtime RFC provider/ecosystem open questions `RFC-OQ-1` through
  `RFC-OQ-9`), and admits non-tiled presentation modes remain a gated host
  surface with no plugin-facing contract yet.
- Capture research `origin/053.md` plugin-generic proposals in the existing
  ecosystem, reuse/provider, IPC-boundary, and UI architecture specifications
  (CTX-0027): private modules versus versioned public services, install versus
  provider requirements, candidate typed adapters and async/stream proxies,
  and a small public Lua SDK with optional replaceable framework plugins.
  Accepted VM isolation, marshalling, synchronous bounded calls, and host
  enforcement remain unchanged. The coverage map records distinct AI/Wheel
  model/tool/agent conclusions as owner-pending; archive recommendation remains
  Partial, not eligible for `.completed`. Documentation capture only.

### Changed

- Reconcile the bundled-plugin split decision with the `file-manager` and
  `git-panel` catalog removals (`bitty` PR #725/`CTX-0399` commit `65aac5c`
  and PR #713/`CTX-0400` commit `e84da34`, both 2026-09-15): both candidates
  move from "split later" to split with catalog entry removed, independent
  package implemented headlessly, and registry entry published
  (`bitty-plugins` PR #23 commit `84f0b7d` and PR #19 commit `9899c1e`);
  panel presentation stays deferred pending the panel-provider contract
  (`bitty-docs` `CTX-0181`, OQ-058).
- Repository metadata baseline: [CONTRIBUTING.md](CONTRIBUTING.md) documents the
  delivery lifecycle, the contributor-branch convention
  (`ctx-XXXX/<type>-<slug>`; external contributors use
  `<handle>/<type>-<slug>`), and the local quality gates;
  [SECURITY.md](SECURITY.md) follows the canonical reporting structure; and
  `.gitattributes` normalizes text files to LF.
