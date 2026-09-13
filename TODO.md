# TODO

Repository work register for `bitty-plugins-docs`. This file stays under 300
lines (enforced by `just agents`); completed items move to git history rather
than accumulating here.

## Current

- [x] Bootstrap repository scaffold (CTX-0187 Phase 1): docs-quality toolchain,
      docs-quality workflow, CarryCtx baseline, documentation skeleton, labels, and
      repository metadata.
- [ ] Phase 3 (separately tracked): migrate the plugin documents from bitty-docs
      (`plugin-roadmap.md`, `plugin-reuse-and-providers.md`, `plugin-platform-rfc.md`,
      and plugin SDK, template, lifecycle, and isolation documents) with history
      preserved, rewriting links and preserving each document's status and
      `website_publish` flag.
- [ ] Later phase: decide and wire submodule composition (`bitty-plugins/docs`)
      if a `bitty-plugins` repository is created.

## Blocked / open

- Content migration is not part of Phase 1. Until migration lands, the tree
  intentionally contains only the documentation map and development workflow.
- The submodule composition decision is deferred behind the `bitty-plugins`
  repository discussion.
