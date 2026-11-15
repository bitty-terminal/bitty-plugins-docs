---
title: Plugin UI Component Catalog (Deprecated)
description: Deprecated earlier composition catalog retained only as a pointer to the extension component catalog
category: extensibility
audience: plugin-author
document_type: specification
status: deprecated
website_publish: false
sidebar_order: 30
---

# Plugin UI Component Catalog (Deprecated)

> Status: **deprecated**. This record's content is superseded by
> [Extension Component Catalog](extension-component-catalog-candidate.md). It
> is retained because published paths and sibling records link to it; it must
> not be read as current advice about the declarative surface.

## Replacement

[Extension Component Catalog (Candidate)](extension-component-catalog-candidate.md)
is the maintained catalog. It carries the same five patterns, adds caller
evidence at `file:line` for every one of them, and states each pattern's
availability against the current build.

| Deprecated entry                | Replacement entry                                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| C-1 edge band                   | [E-1 edge band](extension-component-catalog-candidate.md#e-1-edge-band)                                     |
| C-2 pill row                    | [E-2 pill row](extension-component-catalog-candidate.md#e-2-pill-row)                                       |
| C-3 tooltip and detail fallback | [E-3 tooltip and detail fallback](extension-component-catalog-candidate.md#e-3-tooltip-and-detail-fallback) |
| C-4 modal sheet                 | [E-4 modal sheet](extension-component-catalog-candidate.md#e-4-modal-sheet)                                 |
| C-5 status strip                | [E-5 status strip](extension-component-catalog-candidate.md#e-5-status-strip)                               |

## Why this record is superseded

This revision described the declarative surface as it stood when it was written,
and three shipped changes invalidated its central claims:

- It states that no padding, radius, or border field exists anywhere and lists
  them as future open points. A bounded `style` table (`padding`, `radius`,
  `border`, `border_token`) now ships on the rich vocabulary, and the band path
  deliberately does not consume it.
- It states that there is no hover event and no pointer-enter binding, and builds
  its detail fallback on that absence. Pointer-hover routing (`on_hover` with a
  required region identity and a host-injected `phase`) now ships, and the bar
  plugin drives its tooltip from it.
- It describes band grouping as three anchored rows without naming the `anchor`
  field that implements it, so its band recipe could not be followed.

It also recorded no caller evidence: every recipe was written from the contract
rather than from a shipped composition, so a reader could not tell a real pattern
from a plausible one.

## Deprecation terms

- **Affected versions:** this record as published on 2026-11-03. Its statements
  about padding, radius, border, hover, and group anchoring are wrong for the
  current build; its scene-shape sketches and security review remain broadly
  accurate but are not maintained.
- **Transition:** immediate. Links and cross-references resolve to the
  replacement; no content is maintained here.
- **Removal condition:** this path may be deleted once the replacement is
  `accepted` and no published page, release note, or sibling record references
  it. Until then it stays as a pointer.

## P0 Review Sign-off

Not applicable: this record adds no requirement, capability, or trust decision.
