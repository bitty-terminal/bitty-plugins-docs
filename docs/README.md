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

## Content trees

Canonical plugin-ecosystem documents live in root topic trees grouped by
theme. Each tree has a route-only index; this map links the tree index instead
of duplicating its route table. New trees are added only when real content
lands.

| Tree                                          | Scope and entry point                                                                      |
| --------------------------------------------- | ------------------------------------------------------------------------------------------ |
| [Runtime](../runtime/README.md)               | Plugin host bridge, Lua runtime, sandbox, and isolation and resource ceilings.             |
| [SDK](../sdk/README.md)                       | Public plugin SDK surface: Lua module functions, payloads, and the L1/L2 split.            |
| [Packaging](../packaging/README.md)           | Package integrity, lifecycle, resolver, registry, and provider-ecology contracts.          |
| [Architecture](../architecture/README.md)     | Plugin ecosystem model, out-of-process IPC boundary, and UI extensibility architecture.    |
| [Specifications](../specifications/README.md) | Accepted platform contract and research-type register that stay at the specification root. |
| [Product](../product/README.md)               | Official-status onboarding, split decisions, and plugin roadmap.                           |
| [Extensibility](../extensibility/README.md)   | Pre-implementation plugin-system and package-management contracts.                         |
| `reference/`                                  | Planned; no page until verified implementation evidence lands.                             |

## Process documents

| Document                                                        | Purpose                                                             |
| --------------------------------------------------------------- | ------------------------------------------------------------------- |
| [Development](development/README.md)                            | Contributor entry point and local gates.                            |
| [Documentation workflow](development/documentation-workflow.md) | Normative authoring, metadata, status, and review policy.           |
| [Documentation process TODO](TODO.md)                           | Pointer to the repository work register.                            |
| [Documentation process handoff](HANDOFF.md)                     | Pointer to handoff records for documentation work.                  |
| [Plugin index](plugins/README.md)                               | Registered official plugins, candidate list, and standard page set. |
| [Plugin template](plugins/TEMPLATE.md)                          | Reusable per-plugin page template.                                  |

## Migrated corpus

The plugin-ecosystem corpus migrated from `bitty-docs` at `c664214` with
history preserved (CTX-0001). Root topic trees hold the canonical documents;
each document's frontmatter owns its status.

| Tree                                          | Documents (status)                                                                                                                                                                                                                                |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Runtime](../runtime/README.md)               | [Plugin Host Runtime RFC](../runtime/plugin-host-runtime-rfc.md) (accepted), [Lua Runtime RFC](../runtime/lua-runtime-rfc.md) (accepted), [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md) (accepted).                          |
| [SDK](../sdk/README.md)                       | [Plugin API v1 Lua Surface RFC](../sdk/plugin-api-v1-lua-surface-rfc.md) (accepted).                                                                                                                                                              |
| [Packaging](../packaging/README.md)           | [Package Lifecycle RFC](../packaging/package-lifecycle-rfc.md) (accepted), [Package Follow-up RFC](../packaging/package-followup-rfc.md) (accepted), [Plugin Reuse and Provider Ecology RFC](../packaging/plugin-reuse-and-providers.md) (draft). |
| [Architecture](../architecture/README.md)     | [UI Extensibility Architecture](../architecture/ui-extensibility-architecture.md) (draft), [Plugin Ecosystem Model](../architecture/plugin-ecosystem-model.md) (draft), [Plugin IPC Boundary](../architecture/plugin-ipc-boundary.md) (draft).    |
| [Specifications](../specifications/README.md) | [Plugin Platform RFC](../specifications/plugin-platform-rfc.md) (accepted), [Research 053 and 054 plugin-side conclusions](../specifications/research-053-054-plugin-conclusions.md) (draft).                                                     |
| [Product](../product/README.md)               | [Official plugin onboarding](../product/official-plugin-onboarding.md) (normative), [Plugin Roadmap](../product/plugin-roadmap.md) (draft), [Bundled-Plugin Split Decision (OQ-053)](../product/bundled-plugin-split-decision.md) (accepted).     |
| [Extensibility](../extensibility/README.md)   | [Plugin system](../extensibility/plugin-system.md) (draft), [Plugin package management](../extensibility/package-management.md) (draft).                                                                                                          |

### Research 053 and 054 capture

The [Plugin Ecosystem Model sections 9.6-9.7](../architecture/plugin-ecosystem-model.md#96-four-layer-framework-ecosystem-053-candidate)
record the proposed Rust host / small public Lua SDK / optional framework /
application layering and a full-source coverage/owner-handoff map. Generic
[private-module/public-service contracts](../packaging/plugin-reuse-and-providers.md#cross-package-contracts-053-candidate),
[local/remote async proxy questions](../architecture/plugin-ipc-boundary.md#13-local-and-remote-service-proxies-053-candidate),
and [framework-level UI](../architecture/ui-extensibility-architecture.md#framework-level-lua-ui-053-candidate)
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

- [Runtime index](../runtime/README.md)
- [SDK index](../sdk/README.md)
- [Packaging index](../packaging/README.md)
- [Architecture index](../architecture/README.md)
- [Specifications index](../specifications/README.md)
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
