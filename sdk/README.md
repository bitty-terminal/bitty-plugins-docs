---
title: SDK
description: Index of the public plugin SDK Lua surface contract
category: specifications
audience: plugin-author
document_type: index
status: accepted
website_publish: true
sidebar_order: 12
---

# SDK

Index of the public plugin SDK surface contracts. Normative detail lives in the
linked page; this index carries no duplicate normative prose.

## Admission criteria

An SDK surface contract defines the public module functions, payloads,
compatibility, and versioning for plugin authors. New pages are added only when
real content exists; empty placeholder pages are avoided.

## Authority and status

The table's status column governs each page: the Plugin API v1 Lua Surface RFC
is an accepted contract for OQ-011, while the draft candidate page records
direction only and authorizes no shipped behavior. Acceptance records a
reviewed contract and does not prove implementation. Shared cross-project governance
stays in [bitty-docs](https://github.com/bitty-terminal/bitty-docs) and is
linked, never copied.

## Contract

| Document                                                                      | Status   | Purpose                                                                       |
| ----------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------------- |
| [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md)             | Accepted | Lua module functions, payloads, and the L1/L2 split.                          |
| [bitty.net Lua Request Surface (Candidate)](net-request-surface-candidate.md) | Draft    | Candidate non-blocking `bitty.net` requests over the DIR-030 `net` component. |
