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
      added the `specifications/` register index, moved the research 053/054
      page into the specifications tree with self-contained provenance,
      reconciled the research 053/054 archive-status wording, repointed
      `docs/README.md` to canonical documents, and verified per-directory
      index routes.
- [x] Materialize the thematic topic-tree taxonomy (CTX-0035, Issue #65): moved
      the runtime, SDK, packaging, and architecture contracts out of
      `specifications/` into `runtime/`, `sdk/`, `packaging/`, and
      `architecture/`, added a route-only index per new tree, kept
      `specifications/` for the platform contract and research register,
      updated `docs/README.md`, the tree indexes, and every relative
      cross-reference, and reconciled the `Document form` section with
      bitty-ai-docs.

## Blocked / open

- Classification follow-ups from the plugin migration: `status-system.md` stays
  terminal-owned, and `default-distribution-rfc.md` remains with terminal docs
  pending an ownership decision.
