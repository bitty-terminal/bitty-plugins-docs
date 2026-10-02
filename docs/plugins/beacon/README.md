---
title: Beacon plugin documentation
description: Per-plugin index for the candidate Beacon targeting-policy plugin
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 54
---

# Beacon plugin documentation

Identity, current stage, owning repository, and page links for the candidate
Beacon targeting-policy package.

## Identity

| Field             | Value                                                                                |
| ----------------- | ------------------------------------------------------------------------------------ |
| Plugin id         | `bitty-terminal.beacon` (candidate)                                                  |
| Name              | Beacon                                                                               |
| Package version   | Not created; the repository-creation standard reserves `0.0.1`                       |
| Owning repository | Not created; planned <https://github.com/bitty-terminal/beacon>                      |
| Lua module        | Planned `lua/beacon/`                                                                |
| Capabilities      | Not defined; provider-registration and observation dimensions stay open under OQ-056 |
| Lazy commands     | Not decided; the exact spellings belong to the plugin contract                       |
| Lazy events       | Not decided; the observation names belong to the public host API                     |

## Stage

Candidate direction: no `bitty-terminal/beacon` repository, package, or registry
entry exists. The mechanism/policy split and the Core mechanism names
`TargetEngine` and `AnnotationEngine` are accepted by bitty-docs
[ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md)
(2026-10-03, `W-03`), and the ADR also resolves the extraction scope: policy is
extracted to this optional plugin while the Core mechanism stays in Core. The
plugin repository, this page set, the package, and the actual extraction work
remain direction, not a shipped or implemented plugin. This page set is
documentation only.

## What the plugin is

Beacon is the planned official Lua **policy** layer above the accepted Core
targeting and annotation mechanism. Core owns the always-available
`TargetEngine` and `AnnotationEngine`; Beacon would own policy only: the key
language, which-key and menu grammar, scopes and filters, theme badges, provider
composition, and target-first menus. It is an ordinary optional,
capability-gated package with no private privilege. The Core mechanism works
with zero plugins and `bitty --safe` is unaffected without it.

The plugin-side direction is the
[Beacon Targeting Framework (Candidate)](../../../specifications/beacon-targeting-framework-candidate.md);
the mechanism boundary it sits above is the terminal-side
[Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md).

## Install and usage overview

When the package exists, it would be installed and enabled like any other
official plugin through the registry, with no bundled or private path.
Installation executes no package code, capability grants are explicit and
deny-by-default, and the plugin stays optional. None of this is implemented
today; the creation and registration of the repository is queued (`W-121`, with
`bitty` `CTX-0426`/`CTX-0427` recording the onboarding order).

## Pages

- [Design](design.md) — the Lua policy contract over the Core mechanism.
- [Schemas and contracts](schemas.md) — candidate config, key-language, scope, filter, and badge shapes.
- [Evidence and links](evidence.md) — required evidence and verification placeholders.

## Related

- [Plugin documentation](../README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
- [Plugin Host Runtime RFC](../../../runtime/plugin-host-runtime-rfc.md)
