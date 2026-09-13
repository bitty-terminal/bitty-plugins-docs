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
- [ ] Later phase: decide and wire submodule composition (`bitty-plugins/docs`)
      if a `bitty-plugins` repository is created.

## Blocked / open

- The submodule composition decision is deferred behind the `bitty-plugins`
  repository discussion.
- Classification follow-ups from the plugin migration: `status-system.md` stays
  terminal-owned, and `default-distribution-rfc.md` remains with terminal docs
  pending an ownership decision.
