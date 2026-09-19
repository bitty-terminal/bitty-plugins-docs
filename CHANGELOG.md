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
- Record candidate plugin-ecosystem direction as the draft
  `architecture/plugin-ecosystem-model.md`: plugin taxonomy, platform versus
  extension plugins, extension points, manifest candidates, capability
  non-escalation, extension-platform API versioning, the plugin graph, and
  Panel/Activity implications. Candidate design input, not an accepted
  contract.
- Record candidate out-of-process plugin-boundary direction as the draft
  `architecture/plugin-ipc-boundary.md`:
  out-of-process plugins as a second extension boundary, the plugin event bus,
  the unified Lua/IPC/CLI capability model, capability-token and
  permission-display candidates, crash isolation and supervision questions,
  control-CLI and multi-instance addressing sketches, and the candidate
  three-layer extension framing. Candidate design input, not an accepted
  contract; the diverging `[permissions]` sketch is marked unaccepted.
- Add `docs/plugins/file-manager/` and `docs/plugins/git-panel/` per-plugin
  page sets (CTX-0023, related to Issue #43): README index, design, schemas,
  and evidence for the independent `bitty-terminal.file-manager`
  (observation-only, `terminal.semantic-read`) and `bitty-terminal.git-panel`
  (tiled panel over allowlisted `process.spawn:git` plus accepted
  `[tools.git]` v1) packages; panel presentation deferred pending the
  panel-provider contract. Nothing here is verified or shipped.
- Record candidate plugin-side direction for
  pluggable model access and pluggable pane
  history in
  `packaging/plugin-reuse-and-providers.md` with open-item acceptance
  rows: core keeps the provider contract/registry/routing while vendor
  auth/billing/discovery and the management UI live in plugins with opaque
  credential handles composed with the Secrets direction, and durable history
  becomes an official plugin with append-only compressed segments, storage
  capabilities instead of direct database access, command/output/replay tiers,
  and external history tools as providers and sinks. Candidate design
  input, not an accepted contract; Plugin API v1 defines no `fs.*` Lua entry
  point.
- Disambiguate the `composer` term in the Plugin API v1 Lua Surface RFC and
  record a candidate API stability priority order as a non-normative
  direction in the plugin system extension-levels section (CTX-0025, related
  to Issue #43): the user-facing Command Composer stays a terminal-interaction
  concept owned by `bitty-terminal-docs`, distinct from the internal scene-diff
  step. No normative behavior changes.
- Bridge the candidate ActivityStack direction to the P2 panel-identity
  candidate (CTX-0026, related to Issue #43): the UI extensibility P2 section
  now points plugin authors to the Plugin Ecosystem Model section 9.2 for the
  Panel-is-not-Activity and session-survival direction (acceptance path: the
  Panel Runtime RFC provider/ecosystem open questions `RFC-OQ-1` through
  `RFC-OQ-9`), and admits non-tiled presentation modes remain a gated host
  surface with no plugin-facing contract yet.
- Record candidate plugin-generic proposals in the existing
  ecosystem, reuse/provider, IPC-boundary, and UI architecture specifications
  (CTX-0027): private modules versus versioned public services, install versus
  provider requirements, candidate typed adapters and async/stream proxies,
  and a small public Lua SDK with optional replaceable framework plugins.
  Accepted VM isolation, marshalling, synchronous bounded calls, and host
  enforcement remain unchanged. The coverage map records distinct AI/Wheel
  model/tool/agent conclusions as owner-pending. Documentation direction only.
- Record two candidate specification pages (CTX-0040, Issue #74): the
  [Beacon Targeting Framework (Candidate)](specifications/beacon-targeting-framework-candidate.md)
  for a workspace-wide spatial and semantic targeting engine (target
  references and stale-target validation, the `Action × Target` interaction
  model, provider tiers and cold-path collection, scopes, label allocation,
  the batched annotation layer, the six primitives and session state machine,
  the Core/plugin split, and the mediator security posture), and the
  [Lua UI Component Model (Candidate)](extensibility/lua-ui-component-model-candidate.md)
  for the five-level Lua UI hierarchy, composition from minimal primitives,
  complex widgets as Rust mechanism plus Lua appearance, pluggable theming,
  accessibility roles, and the candidate panel service model
  (`PanelProvider`, `ActivityStack`, decoupled lifecycle, attention requests,
  chrome slots, and rule requests). Every direction is marked
  Accepted/Candidate/Owner-pending/Open; the terminal-side core engine, the
  terminal-side UI runtime candidate, and the OQ-052/OQ-056/OQ-058/OQ-088/OQ-089
  decisions remain owner-pending. Draft candidate direction only; no accepted
  interface and no implementation claim.
- Record the Phodopus plugin-runtime candidate (CTX-0041, Issue #76): the
  [Phodopus Plugin Runtime (Candidate)](runtime/phodopus-runtime-candidate.md)
  captures the plugin-side consequences of adopting Phodopus (a sandbox-first
  successor fork of Piccolo) — the `require` searcher chain with the
  `PluginVfsSearcher`/`LuxPackageSearcher` capability bindings, hard builder
  memory and Fuel quotas, authentic Lua pattern semantics versus Rust `regex`,
  `utf8.*` code points versus terminal typography, the runtime-agnostic
  `HostOp::Pending` async bridge with an optional Tokio adapter, the
  generic-runtime versus `bitty-lua` boundary, and the six-phase roadmap.
  Refinement pointer sentences are added to the accepted
  [Lua Runtime RFC](runtime/lua-runtime-rfc.md) and
  [Isolation and Resource RFC](runtime/isolation-resource-rfc.md) without
  changing their frontmatter, status, or accepted claims. The `bitty-lua`
  implementation is deferred until Phodopus is usable; today's accepted
  `mlua`/Lua 5.4 and `piccolo 0.3.3` contract is unchanged. Draft candidate
  direction only; no accepted interface and no implementation claim.

### Changed

- Consolidate the diagram suite under the canonical architecture tree (CTX-0039,
  Issue #72): move `docs/architecture/` (route index, `glossary.yaml`, the
  Mermaid sources, and the generated SVG exports) to `architecture/diagrams/`
  with `git mv`; index the diagram subtree from `architecture/README.md` and the
  repository and documentation maps; update the glossary header and the `mmdc`
  regeneration command; and validate every SVG through `git ls-files` instead
  of a `docs/`-only directory scan. Diagram content, frontmatter, and statuses
  are unchanged.
- Make the plugin corpus research-free (CTX-0037, Issue #68): add the normative
  `Docs self-containment` section to
  `docs/development/documentation-workflow.md`; convert the
  plugin-conclusion page into the self-contained
  `specifications/plugin-contract-direction.md`; strip record numbers, archive
  paths and status, and archive wording from the architecture, packaging,
  product, extensibility, SDK, and index pages and from the
  `architecture/diagrams/glossary.yaml` canonical inventory while preserving
  every claim, status, and candidate qualifier; rename the affected anchors; and
  update every cross-reference. Technical claims, draft status, and
  owner-pending pointers are unchanged.
- Materialize the approved thematic topic-tree taxonomy (CTX-0035, Issue #65):
  move the plugin-host runtime, Lua runtime, and isolation and resource
  contracts to `runtime/`; the Plugin API v1 Lua surface contract to `sdk/`;
  the package lifecycle, package follow-up, and reuse/provider contracts to
  `packaging/`; and the ecosystem model, IPC boundary, and UI extensibility
  contracts to `architecture/`. `specifications/` keeps the accepted Plugin
  Platform RFC and the candidate-direction register. Each new tree
  gains a route-only index, the maps and every relative cross-reference are
  updated, and the `Document form` section is aligned with bitty-ai-docs.
  Links only; no document status, claim, or `website_publish` flag changes.
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
