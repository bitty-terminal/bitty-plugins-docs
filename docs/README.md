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
plugin ecosystem. The repository is in its bootstrap state (CTX-0187 Phase 1):
the docs-quality toolchain and this skeleton exist, while plugin documents are
migrated from `bitty-docs` in a later, separately tracked phase. Nothing on
this page claims migrated content.

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

The plugin-ecosystem documents migrate into topic trees:

| Planned tree        | Content                                                       |
| ------------------- | ------------------------------------------------------------- |
| `sdk/`              | Public plugin API surface, versioning, and compatibility.     |
| `manifests/`        | Manifest fields, schemas, and validation contracts.           |
| `lifecycle/`        | Install, activation, update, disable, and removal semantics.  |
| `isolation/`        | Capability model, resource limits, and failure semantics.     |
| `specifications/`   | Versioned technical contracts with verification obligations.  |
| `reference/`        | Factual lookup material derived from implementation evidence. |
| `plugins/<plugin>/` | Per-plugin pages using the standard page set.                 |

Trees are created only as real content lands; empty placeholder pages are not
added.

## Maintaining the corpus

1. Update the canonical topic document first.
2. Keep status labels honest: `draft`, `accepted`, `normative`, `stable`,
   `deprecated`, `archived`.
3. Cross-link one authoritative definition instead of copying divergent
   wording.
4. Update this index and the root `README.md` when navigation changes.
5. Run `just check` before every push; documentation synchronization is part of
   delivery completion.
