---
title: Statusline plugin design
description: Scope presentation policy and capability design for the statusline plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 39
---

# Statusline plugin design

## Problem

Users need terminal and workspace state (working directory, mode, Git state,
task state) presented compactly without a plugin-side shell prompt takeover.

## Scope

In scope: composition of declarative fragments into the statusline/workspaceline
slot from read-only terminal state, recomposed when the declared observation
events fire.

Out of scope: terminal truth, input, parser, and render hot paths; prompt
replacement. The statusline is terminal-owned chrome and does not absorb
shell-prompt ownership, so it is not a starship replacement. Whether it should
eventually carry prompt-class presentation is an open design question recorded
by the roadmap, not accepted here.

## Capability and trust boundaries

- Requests `terminal.semantic-read` (gates `bitty.terminal.snapshot`, the
  read-only committed-state observation the statusline composes from) and
  `ui.rich` (gates `bitty.ui.mount`/`bitty.ui.update` on the `statusline`
  slot). Both identifiers are unchanged from the bundled manifest.
- Deny by default: no filesystem, process, network, clipboard, or
  terminal-input authority is requested.

## Failure behavior

The plugin has no commands; the manifest-declared observation events activate
and drive recomposition. Missing semantic zones degrade the presented fields
rather than failing activation.

## Alternatives considered

A bundled-only statusline was rejected by the
[bundled plugin split decision](../../../product/bundled-plugin-split-decision.md),
which moved it to an independent first-party package.

## Related

- [Statusline plugin documentation](README.md)
- [Plugin Roadmap](../../../product/plugin-roadmap.md) (draft)
