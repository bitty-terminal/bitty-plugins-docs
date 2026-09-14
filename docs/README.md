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
- The repository is designed to be mounted at `bitty-plugins/docs` as a Git
  submodule if a `bitty-plugins` repository is created; that composition is
  deferred. The SDK and template remain independent repositories.

## Current tree

| Document                                                        | Purpose                                                   |
| --------------------------------------------------------------- | --------------------------------------------------------- |
| [Development](development/README.md)                            | Contributor entry point and local gates.                  |
| [Documentation workflow](development/documentation-workflow.md) | Normative authoring, metadata, status, and review policy. |
| [Plugin index](plugins/README.md)                               | Per-plugin candidate list and standard page set.          |
| [Plugin template](plugins/TEMPLATE.md)                          | Reusable per-plugin page template.                        |

## Migrated corpus

The plugin-ecosystem corpus migrated from `bitty-docs` at `c664214` with
history preserved (CTX-0001). Root topic trees hold the canonical documents;
each document's frontmatter owns its status.

| Document                                                                                 | Status   | Purpose                                                        |
| ---------------------------------------------------------------------------------------- | -------- | -------------------------------------------------------------- |
| [Plugin Platform RFC](../specifications/plugin-platform-rfc.md)                          | accepted | API v1 surface, capability and manifest model, event pipeline. |
| [Plugin Host Runtime RFC](../specifications/plugin-host-runtime-rfc.md)                  | accepted | Runtime host bridge, per-plugin VM lifecycle, host services.   |
| [Plugin API v1 Lua Surface RFC](../specifications/plugin-api-v1-lua-surface-rfc.md)      | accepted | Lua module functions, payloads, and L1/L2 split.               |
| [Plugin Reuse and Provider Ecology RFC](../specifications/plugin-reuse-and-providers.md) | draft    | Post-1.0 reuse principle and provider ecology.                 |
| [Lua Runtime RFC](../specifications/lua-runtime-rfc.md)                                  | accepted | Lua runtime, sandbox, standard-library subset, modules.        |
| [Isolation and Resource RFC](../specifications/isolation-resource-rfc.md)                | accepted | Isolation boundaries, resource ceilings, failure semantics.    |
| [Package Lifecycle RFC](../specifications/package-lifecycle-rfc.md)                      | accepted | Integrity chain, staged activation, and rollback.              |
| [Package Follow-up RFC](../specifications/package-followup-rfc.md)                       | accepted | Resolver, yank, prerelease, registry, key management.          |
| [UI Extensibility Architecture](../specifications/ui-extensibility-architecture.md)      | draft    | UI extension points, ownership boundaries, Lua surface.        |
| [Plugin Roadmap](../product/plugin-roadmap.md)                                           | draft    | First-party and featured plugin sequencing.                    |
| [Bundled-Plugin Split Decision (OQ-053)](../product/bundled-plugin-split-decision.md)    | accepted | Per-candidate split/stay-bundled verdicts and their gates.     |
| [Plugin system](../extensibility/plugin-system.md)                                       | draft    | Plugin boundaries, isolation, composition, and lifecycle.      |
| [Plugin package management](../extensibility/package-management.md)                      | draft    | Manifests, sources, updates, rollback, and trust.              |

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

## Planned structure

The initial migration covers `specifications/`, `product/`, and
`extensibility/` plus the per-plugin pages. Remaining trees are added only as
real content lands; empty placeholder pages are not added.

| Tree                     | Content                                                       |
| ------------------------ | ------------------------------------------------------------- |
| `sdk/`                   | Public plugin API surface, versioning, and compatibility.     |
| `manifests/`             | Manifest fields, schemas, and validation contracts.           |
| `lifecycle/`             | Install, activation, update, disable, and removal semantics.  |
| `isolation/`             | Capability model, resource limits, and failure semantics.     |
| `reference/`             | Factual lookup material derived from implementation evidence. |
| `docs/plugins/<plugin>/` | Per-plugin pages using the standard page set.                 |

## Maintaining the corpus

1. Update the canonical topic document first.
2. Keep status labels honest: `draft`, `accepted`, `normative`, `stable`,
   `deprecated`, `archived`.
3. Cross-link one authoritative definition instead of copying divergent
   wording.
4. Update this index and the root `README.md` when navigation changes.
5. Run `just check` before every push; documentation synchronization is part of
   delivery completion.
