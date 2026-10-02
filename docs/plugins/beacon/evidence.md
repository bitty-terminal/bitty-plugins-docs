---
title: Beacon plugin evidence and links
description: Required evidence and verification placeholders for the candidate Beacon targeting-policy plugin
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 57
---

# Beacon plugin evidence and links

## Tasks and provenance

- [ADR 0018](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0018-beacon-mechanism-policy-split.md)
  (accepted 2026-10-03, `W-03`) accepted the mechanism/policy split, named the
  Core mechanism `TargetEngine`/`AnnotationEngine`, and resolved the extraction
  scope: policy moves to the optional `beacon` plugin, the mechanism stays in
  Core for 0.1.0.
- The terminal-side
  [Beacon Core Mechanism Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/beacon-core-mechanism-contract.md)
  (`W-81`) defines the mechanism boundary this plugin policy sits above.
- The
  [Beacon Targeting Framework (Candidate)](../../../specifications/beacon-targeting-framework-candidate.md)
  records the plugin-side direction; its `B-8` open points 1 and 3 are decided
  by ADR 0018 and synchronized under `W-12`.
- `bitty-plugins-docs` `CTX-0069` (`W-90`, Issue #128) — this plugin page set;
  `W-91` reconciles it with the actual SDK overlay and input-capture surface.
- `bitty` `W-29` (host API) and `W-120` (SDK surface) own the public spellings
  and capability dimensions this policy depends on; `W-102` extracts the policy
  from Core while retaining the mechanism.
- Repository creation and registration are queued as `W-121`; `bitty`
  `CTX-0426`/`CTX-0427` record the onboarding order in the
  [official plugin onboarding](../../../product/official-plugin-onboarding.md).
- The `bar` plugin (`W-53`) is a downstream consumer of the same
  workspace-wide targeting framework.

## Required evidence

Each item must be produced by the owning implementation task before any
promotion beyond candidate. Nothing here is a result; every state is **Not
produced**.

| Requirement           | Required evidence artifact                                                                         | State        |
| --------------------- | -------------------------------------------------------------------------------------------------- | ------------ |
| Capability gating     | Host integration test proving a denied grant fails closed before any side effect.                  | Not produced |
| No private bypass     | Parity test proving an equivalent third-party plugin reaches the same public surfaces.             | Not produced |
| Safe mode             | `bitty --safe` starts with zero third-party plugins and targeting still works.                     | Not produced |
| No hot-path callback  | Test proving capture is transient and revocable and no plugin callback runs on the input hot path. | Not produced |
| Fail-closed handles   | Test proving an invalidated or re-registered target fails closed with `StaleTarget`.               | Not produced |
| No Event-Bus exposure | Test proving target and annotation internals are never published on the Event Bus.                 | Not produced |
| Policy optionality    | Test proving disabling or removing the plugin removes policy only.                                 | Not produced |
| Public metadata only  | Review confirming no private plugin state is reachable through provider metadata.                  | Not produced |
| Documentation gates   | Repository-local `just check` with zero issues for each owning repository.                         | Not produced |

## Verification placeholders

- `W-121` owns the package, manifest, and Lua policy tests; the manifest and
  package evidence belong to the created `bitty-terminal/beacon` repository.
- `W-102` owns the Core policy-retirement evidence while retaining the
  mechanism.
- `W-29` owns the host API evidence the integration tests run against; `W-120`
  owns the SDK surface evidence.
- Security review evidence for the capability, capture, dispatch, and
  metadata boundaries is required before promotion, and the P0 controls are not
  weakened by this page set.

## Status

Candidate direction: the page set is documentation only, and no
`bitty-terminal/beacon` repository, package, manifest, or registry entry
exists. Not implemented, verified, compatible, or shipped.

## Related

- [Beacon plugin documentation](README.md)
- [Beacon plugin design](design.md)
- [Beacon plugin schemas and contracts](schemas.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
- [Official plugin onboarding](../../../product/official-plugin-onboarding.md)
- [Bundled-plugin split decision](../../../product/bundled-plugin-split-decision.md)
