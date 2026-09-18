---
title: Specifications
description: Index of accepted and draft plugin-ecosystem technical contracts with their open-question scope and status
category: specifications
audience: contributor
document_type: index
status: accepted
website_publish: true
sidebar_order: 10
---

# Specifications

This register indexes the versioned technical contracts that govern the Bitty
plugin ecosystem. Acceptance records a reviewed contract; it does not prove
implementation, and each document's own evidence rules still apply. Draft work
is listed separately and does not authorize shipped, stable, normative, or
compatibility-guaranteed behavior.

## Accepted specifications

| Specification                                                                                | Closes                                                                      | Status   |
| -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | -------- |
| [Plugin Platform RFC](plugin-platform-rfc.md)                                                | OQ-011, OQ-012, OQ-013                                                      | Accepted |
| [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md)                            | OQ-011 (v1 surface spelling refinement)                                     | Accepted |
| [Plugin Host Runtime RFC](plugin-host-runtime-rfc.md)                                        | OQ-033, OQ-034, OQ-035                                                      | Accepted |
| [Lua Runtime RFC](lua-runtime-rfc.md)                                                        | OQ-009                                                                      | Accepted |
| [Isolation and Resource RFC](isolation-resource-rfc.md)                                      | OQ-014                                                                      | Accepted |
| [Package integrity, activation, and rollback](package-lifecycle-rfc.md)                      | OQ-021 (partially OQ-022; residual items migrated to OQ-026 through OQ-029) | Accepted |
| [Package Resolver, Version Lifecycle, Registry, and Key Management](package-followup-rfc.md) | OQ-022, OQ-026, OQ-027, OQ-028, OQ-029                                      | Accepted |

## Draft specifications

| Specification                                                          | Targets                                                                        | Status |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------ |
| [Plugin Reuse and Provider Ecology RFC](plugin-reuse-and-providers.md) | OQ-011, OQ-012, OQ-013 (provider-ecology follow-up)                            | Draft  |
| [UI Extensibility Architecture](ui-extensibility-architecture.md)      | OQ-043, OQ-044, OQ-049 (OQ-041 and OQ-045 accepted in RFC-0001)                | Draft  |
| [Plugin Ecosystem Model](plugin-ecosystem-model.md)                    | — (research-derived design input from records 039, 040, and 053; no OQ)        | Draft  |
| [Plugin IPC Boundary](plugin-ipc-boundary.md)                          | — (research-derived design input from record 041; aligns with accepted OQ-018) | Draft  |

Acceptance for these drafts requires independent category-owner, docs-curator,
and security-reviewer evidence before the status changes.

## Research-type specifications

| Specification                                                                          | Provenance                                                                        | Status |
| -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ------ |
| [Research 053 and 054 plugin-side conclusions](research-053-054-plugin-conclusions.md) | Research records 053 and 054 (plugin-side conclusions; draft distillation, no OQ) | Draft  |

Research-type pages preserve provenance and observations; they never become a
decision or an implementation claim by implication.

## Admission criteria

A specification defines boundaries, inputs, outputs, invariants, errors,
resource limits, compatibility, lifecycle, recovery, and verification. It links
the requirements and decisions it implements and includes security review where
trust boundaries are involved. New pages are added only when real content
exists; empty placeholder specifications are avoided.

## Authority and status

A `normative` specification governs its declared version and scope. Draft text
does not authorize shipped, stable, normative, or compatibility-guaranteed
behavior and does not form public reference. The lifecycle is
`Draft -> experimental review evidence -> Accepted -> normative`; only
`Accepted` or `normative` documents authorize shipped behavior. Shared
cross-project decisions and open questions live in
[bitty-docs](https://github.com/bitty-terminal/bitty-docs) and are linked,
never copied.
