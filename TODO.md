# TODO

Repository work register for `bitty-plugins-docs`. This file stays under 300
lines (enforced by `just agents`); completed items move to git history rather
than accumulating here.

## Current

- [x] Bootstrap repository scaffold (CTX-0187 Phase 1): docs-quality toolchain,
      docs-quality workflow, CarryCtx baseline, documentation skeleton, labels, and
      repository metadata.
- [x] Plugin-ecosystem migration (CTX-0001, parent bitty-docs CTX-0187 Phase 3):
      fourteen documents imported from `bitty-docs` `c664214` with history,
      rewritten cross-repository links, and preserved status and
      `website_publish` flags.
- [x] Submodule composition (`bitty-plugins/docs`, CTX-0006): `bitty-plugins`
      exists with `sdk`/`template`/`docs` submodules and
      `plugins/activity`+`palette`+`statusline`; this repository is mounted at
      `bitty-plugins/docs` (branch `main`).
- [x] Structural alignment to the terminal-docs standard (CTX-0033, Issue #61):
      added the `specifications/` register index, moved the plugin-conclusion
      page into the specifications tree with self-contained wording,
      reconciled its archive-status wording, repointed
      `docs/README.md` to canonical documents, and verified per-directory
      index routes.
- [x] Materialize the thematic topic-tree taxonomy (CTX-0035, Issue #65): moved
      the runtime, SDK, packaging, and architecture contracts out of
      `specifications/` into `runtime/`, `sdk/`, `packaging/`, and
      `architecture/`, added a route-only index per new tree, kept
      `specifications/` for the platform contract and candidate register,
      updated `docs/README.md`, the tree indexes, and every relative
      cross-reference, and reconciled the `Document form` section with
      bitty-ai-docs.
- [x] Make the plugin corpus research-free (CTX-0037, Issue #68): added the
      normative `Docs self-containment` section to
      `docs/development/documentation-workflow.md`; converted the
      plugin-conclusion page into the self-contained
      `specifications/plugin-contract-direction.md`; stripped record numbers,
      archive paths/status, and archive wording from the architecture,
      packaging, product, extensibility, SDK, and index pages; fixed the
      folded-in audit defects; and updated every cross-reference and anchor.
- [x] Consolidate the diagram suite under the canonical architecture tree
      (CTX-0039, Issue #72): moved `docs/architecture/` to
      `architecture/diagrams/`, indexed the subtree from `architecture/README.md`
      and the repository and documentation maps, updated the glossary header and
      the `mmdc` regeneration command, switched the SVG gate to `git ls-files`,
      and left `docs/` holding only repository process documents.
- [x] Record the Beacon targeting and Lua UI component candidates (CTX-0040,
      Issue #74): added
      `specifications/beacon-targeting-framework-candidate.md` and
      `extensibility/lua-ui-component-model-candidate.md`, registered both in
      the topic-tree indexes and `docs/README.md`, and recorded the
      terminal-side core engine, the terminal-side UI runtime candidate, and
      the OQ-052/OQ-056/OQ-058/OQ-088/OQ-089 decisions as owner-pending.

## Blocked / open

- Classification follow-ups from the plugin migration: `status-system.md` stays
  terminal-owned, and `default-distribution-rfc.md` remains with terminal docs
  pending an ownership decision.
