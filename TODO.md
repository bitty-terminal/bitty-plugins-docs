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

## Blocked / open

- Classification follow-ups from the plugin migration: `status-system.md` stays
  terminal-owned, and `default-distribution-rfc.md` remains with terminal docs
  pending an ownership decision.
