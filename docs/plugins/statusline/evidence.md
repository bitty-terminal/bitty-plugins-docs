---
title: Statusline plugin evidence and links
description: Archived statusline source identity disposition and verification backlog
category: project
audience: plugin-author
document_type: register
status: draft
website_publish: false
sidebar_order: 41
---

# Statusline plugin evidence and links

## Purpose

Record the archived statusline source identity and the evidence this page set
does and does not carry, without implying eligibility, verification, or a live
presentation claim.

## Source identity

The source is `statusline` `2b9973c7569398437ba632b41cce1fa37577563c`, archived
on GitHub (last push 2026-10-02). Package version is `0.0.1` with Core
`>=0.1,<1.0` and Plugin API `^1.0` declarations. There is no
`registry/official/statusline.toml` entry in `bitty-plugins`, so the package is
not published through the official registry.

Historical context:

- [Bundled plugin split decision](../../../product/bundled-plugin-split-decision.md)
  (OQ-053) moved the statusline out of the bundled catalog into the independent
  first-party package `bitty-terminal/statusline`.
- `bitty` `CTX-0398` — owning task for the statusline split.
- `bitty-plugins-docs` `CTX-0004` — this per-plugin page set; `CTX-0002`
  recorded the statusline split implementation status.
- Scaffolded from
  [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template).

## Disposition

Implemented in the archived source: declarative `Row`/`Text` composition under
`ui.rich`, driven by the four declared observation events, with guarded reads of
six formatting keys that the manifest never declared. On a strict host those
reads are denied and local defaults serve. The repository is archived, so no
active owner can add the declarations; the successor consolidated presentation
is described by the bar manifest and is a separate contract. Nothing here is
verified, compatible, or shipped.

## Verification backlog

The owning repository provided its own `just check` gates, headless manifest and
package validation, LuaLS conformance, and `bitty-plugin-lint`; this page set
does not restate that evidence and does not claim independent verification.

Open work, with no active owner while the repository stays archived:

- Declare the six formatting keys if the surface is revived, or state successor
  keys on whichever package carries statusline presentation.
- Record any eligibility decision for a `>=0.1,<1.0` range against a real host
  release.
- Provide end-to-end presentation evidence before any usability claim.

### References

- [Statusline plugin documentation](README.md)
- [Bar plugin documentation](../bar/README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
