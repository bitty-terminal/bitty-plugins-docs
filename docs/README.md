---
title: Documentation map
description: Canonical navigation and authority rules for the Bitty plugin ecosystem documentation corpus
category: project
audience: mixed
document_type: index
status: accepted
website_publish: true
sidebar_order: 1
---

# Documentation map

This index is the entry point for the canonical documentation of the Bitty
plugin ecosystem. The corpus migrated from `bitty-docs` at `c664214` (CTX-0001,
parent bitty-docs CTX-0187) and lives in root topic trees plus the per-plugin
page sets under `docs/plugins/`. Migrated documents keep their own status; this
map does not upgrade any claim.

## Authority and composition

- This repository owns plugin-ecosystem documentation: SDK, manifests,
  lifecycle, isolation and capabilities, and per-plugin design notes.
- Shared cross-project governance lives in
  [bitty-docs](https://github.com/bitty-terminal/bitty-docs): decisions, the
  security corpus, sources, findings, reviews, handoff, project state, roadmap,
  and releases. This repository links to those documents instead of copying
  them.
- Sibling documentation repositories:
  [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
  (terminal platform) and
  [bitty-ai-docs](https://github.com/bitty-terminal/bitty-ai-docs) (AI core).
- This repository is mounted at `bitty-plugins/docs` as a Git submodule
  (branch `main`) of
  [bitty-plugins](https://github.com/bitty-terminal/bitty-plugins), which holds
  the plugin registry and the official plugins as pinned submodules. The SDK
  and template remain independent repositories.

## Current tree

| Document                                                        | Purpose                                                             |
| --------------------------------------------------------------- | ------------------------------------------------------------------- |
| [Development](development/README.md)                            | Contributor entry point and local gates.                            |
| [Documentation workflow](development/documentation-workflow.md) | Normative authoring, metadata, status, and review policy.           |
| [Documentation process TODO](TODO.md)                           | Pointer to the repository work register.                            |
| [Documentation process handoff](HANDOFF.md)                     | Pointer to handoff records for documentation work.                  |
| [Plugin index](plugins/README.md)                               | Registered official plugins, candidate list, and standard page set. |
| [Plugin template](plugins/TEMPLATE.md)                          | Reusable per-plugin page template.                                  |
| [Specification register](../specifications/README.md)           | Accepted and draft plugin-ecosystem contracts with their OQ scope.  |

## Migrated corpus

The plugin-ecosystem corpus migrated from `bitty-docs` at `c664214` with
history preserved (CTX-0001). Root topic trees hold the canonical documents;
each document's frontmatter owns its status.

| Document                                                                                 | Status    | Purpose                                                                |
| ---------------------------------------------------------------------------------------- | --------- | ---------------------------------------------------------------------- |
| [Plugin Platform RFC](../specifications/plugin-platform-rfc.md)                          | accepted  | API v1 surface, capability and manifest model, event pipeline.         |
| [Plugin Host Runtime RFC](../specifications/plugin-host-runtime-rfc.md)                  | accepted  | Runtime host bridge, per-plugin VM lifecycle, host services.           |
| [Plugin API v1 Lua Surface RFC](../specifications/plugin-api-v1-lua-surface-rfc.md)      | accepted  | Lua module functions, payloads, and L1/L2 split.                       |
| [Plugin Reuse and Provider Ecology RFC](../specifications/plugin-reuse-and-providers.md) | draft     | Post-1.0 reuse principle and provider ecology.                         |
| [Lua Runtime RFC](../specifications/lua-runtime-rfc.md)                                  | accepted  | Lua runtime, sandbox, standard-library subset, modules.                |
| [Isolation and Resource RFC](../specifications/isolation-resource-rfc.md)                | accepted  | Isolation boundaries, resource ceilings, failure semantics.            |
| [Package Lifecycle RFC](../specifications/package-lifecycle-rfc.md)                      | accepted  | Integrity chain, staged activation, and rollback.                      |
| [Package Follow-up RFC](../specifications/package-followup-rfc.md)                       | accepted  | Resolver, yank, prerelease, registry, key management.                  |
| [UI Extensibility Architecture](../specifications/ui-extensibility-architecture.md)      | draft     | UI extension points, ownership boundaries, Lua surface.                |
| [Plugin Ecosystem Model](../specifications/plugin-ecosystem-model.md)                    | draft     | Research-derived plugin taxonomy and Panel-as-host direction.          |
| [Plugin IPC Boundary](../specifications/plugin-ipc-boundary.md)                          | draft     | Research-derived out-of-process boundary and unified capability model. |
| [Official plugin onboarding](../product/official-plugin-onboarding.md)                   | normative | Official-status entry criteria and registration policy.                |
| [Plugin Roadmap](../product/plugin-roadmap.md)                                           | draft     | First-party and featured plugin sequencing.                            |
| [Bundled-Plugin Split Decision (OQ-053)](../product/bundled-plugin-split-decision.md)    | accepted  | Per-candidate split/stay-bundled verdicts and their gates.             |
| [Plugin system](../extensibility/plugin-system.md)                                       | draft     | Plugin boundaries, isolation, composition, capabilities, lifecycle.    |
| [Plugin package management](../extensibility/package-management.md)                      | draft     | Manifests, sources, updates, rollback, and trust.                      |

### Research 053 and 054 capture

The [Plugin Ecosystem Model sections 9.6-9.7](../specifications/plugin-ecosystem-model.md#96-four-layer-framework-ecosystem-053-candidate)
record the proposed Rust host / small public Lua SDK / optional framework /
application layering and a full-source coverage/owner-handoff map. Generic
[private-module/public-service contracts](../specifications/plugin-reuse-and-providers.md#cross-package-contracts-053-candidate),
[local/remote async proxy questions](../specifications/plugin-ipc-boundary.md#13-local-and-remote-service-proxies-053-candidate),
and [framework-level UI](../specifications/ui-extensibility-architecture.md#framework-level-lua-ui-053-candidate)
are captured as proposals against unchanged accepted boundaries. The research
archive marked records 053 and 054 Captured and renamed their originals with a
`.completed` suffix on 2026-09-18 under an owner directive that makes the
`*-docs` corpora the working corpus; that archive-level rename does not assert
owner-verified capture here, and distinct AI/Wheel model, tool, and agent
conclusions remain owner-pending. This capture accepts no APIs and claims no
implementation. A complementary single-entry distillation of the 053 and 054
plugin-side conclusions is recorded in
[Research 053 and 054 plugin-side conclusions](../specifications/research-053-054-plugin-conclusions.md).

## Per-plugin standard page set

Each documented plugin gets `docs/plugins/<plugin>/` following the standard page
set. The set separates candidate intent, accepted contracts, and evidence so no
page implies shipped behavior it cannot support.

| Page          | Typical `document_type`   | Purpose                                                              |
| ------------- | ------------------------- | -------------------------------------------------------------------- |
| `README.md`   | `index`                   | Identity, current stage, owning repository, and page links.          |
| `design.md`   | `specification`           | Scope, UX, capability boundaries, and mechanism/policy split.        |
| `schemas.md`  | `contract` or `reference` | Manifest fields, configuration keys, wire/API schemas, and versions. |
| `evidence.md` | `register`                | Decision links, experiments, reviews, and test/release evidence.     |

Rules:

- Cross-project contracts and registers stay in the shared directories; a
  plugin page links to them instead of restating them.
- Use only the allowed metadata values; "candidate" and "planned" are prose,
  not an implementation claim.
- Create only pages that have real content; empty placeholder pages are avoided
  so the tree does not imply work that has not happened.

## Related

- [Extensibility index](../extensibility/README.md)
- [Product index](../product/README.md)

## Maintaining the corpus

1. Update the canonical topic document first.
2. Keep status labels honest: `draft`, `accepted`, `normative`, `stable`,
   `deprecated`, `archived`.
3. Cross-link one authoritative definition instead of copying divergent
   wording.
4. Update this index and the root `README.md` when navigation changes.
5. Run `just check` before every push; documentation synchronization is part of
   delivery completion.
