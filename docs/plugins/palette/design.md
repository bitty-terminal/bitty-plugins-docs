---
title: Palette plugin design
description: Scope overlay presentation and capability design for the palette plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 35
---

# Palette plugin design

## Problem

Users need a keyboard-driven command and picker surface without leaving the
terminal workspace.

## Scope

In scope: an overlay presenting declarative lists and text primitives driven by
the plugin's toggle command and focus changes.

Out of scope: terminal truth, input, parser, and render hot paths; the palette
renders through the accepted rich/overlay presentation surface only and adds
no authority beyond its declared capabilities.

## Capability and trust boundaries

- Requests `ui.rich` (gates `bitty.ui.mount`/`bitty.ui.update`) and
  `ui.overlay` (the overlay slot).
- The bundled Rust realization declared only `ui.overlay` because it used the
  lower-level Panel Runtime overlay path; the accepted Plugin API v1 Lua
  overlay path requires both. This intentional difference is recorded in the
  owning repository manifest and the
  [Plugin API v1 Lua Surface RFC](../../../specifications/plugin-api-v1-lua-surface-rfc.md).
- Deny by default: no allow-all identifier, and no filesystem, process,
  network, clipboard, or terminal-input authority is requested.

## Failure behavior

The plugin registers statically through lazy triggers and creates no VM until
the toggle command or `focus.changed` fires. Overlay availability is host
owned; absence degrades to no palette rather than a broken panel.

## Alternatives considered

A bundled-only palette was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved the palette to an independent first-party package.

## Related

- [Palette plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
