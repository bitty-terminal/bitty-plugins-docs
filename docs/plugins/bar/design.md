---
title: Bar plugin design
description: Scope presentation policy and capability design for the unified bar plugin
category: project
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 51
---

# Bar plugin design

## Problem

Rendering separate tabline and statusline plugins wastes two vertical rows of
terminal grid space, which constitutes a significant loss on compact displays
(such as standard 24-row or 30-row terminal windows). Furthermore, users
accustomed to modern tiling window managers (such as Hyprland, Sway, or i3)
expect sparse, on-demand workspace tags (`1, 2, 4, 7, 9`) and system context
displayed together in a single non-occluding edge band similar to Waybar.

## Scope

In scope:

- Composition of a single 1-row edge-band presentation mounted in the
  `ui.chrome-band` slot (`STATUS_BAR_ROWS = 1`, configurable at top or bottom).
- Modular 3-zone layout:
  - Left: sparse workspace pills (active, inactive, urgent states), optional
    add-workspace pill (`+`).
  - Center: active pane metadata (process name, window title, truncated working
    directory).
  - Right: background agent indicators (active headless panels), system
    metrics, and clock.
- Pointer interaction:
  - Left-clicking a workspace pill dispatches `workspace_focus:N`.
  - Left-clicking `+` dispatches `workspace_new`.
  - Middle-clicking a workspace pill dispatches `workspace_close_request:N`.
  - Mouse wheel scrolling over the workspace section cycles through active
    sparse workspaces in numerical order.

Out of scope:

- Terminal truth, input handling, and VT parser hot paths.
- Shell prompt replacement: the bar is terminal-owned chrome and does not
  absorb shell prompt responsibilities (it is not a starship replacement).
- Core workspace lifecycle and storage: Core owns workspace allocation,
  on-demand creation, process tree attachment, and prune-on-empty cleanup per
  [ADR 0014](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0014-workspace-core-presentation-plugins.md).

## Capability and trust boundaries

- Requests `ui.chrome-band` (gates mounting and updating the reserved 1-row
  chrome band) and `terminal.semantic-read` (gates read-only observation of
  committed workspace and terminal metadata).
- Deny by default: no filesystem write, no process spawn, no network, no
  clipboard authority, and no terminal input injection authority is requested.
- Presentation only: plugins alter presentation, never terminal truth. The bar
  consumes state snapshots and events asynchronously and never blocks the input,
  parser, or render hot paths.

## Mechanism and policy split

Under ADR-0014, Bitty strictly separates mechanism from presentation:

1. **Core mechanism**: Core provides sparse workspace addressing
   (`WorkspaceId(u64)`), switch-or-create semantics, prune-on-empty lifecycle,
   headless agent panel tracking, and zero-occlusion layout partitioning via
   `ChromeInsets`.
2. **Plugin policy**: The Bar plugin defines the visual arrangement, module
   configuration, theme token mapping, pill formatting, and pointer event
   translation to Core workspace commands.

## Failure behavior

Missing semantic zones or unavailable system statistics degrade gracefully to
fallback text rather than aborting presentation. Lua runtime errors inside the
bar plugin are caught and isolated by the plugin host VM, leaving terminal grid
rendering and PTY sessions intact.

## Alternatives considered

- Retaining separate `tabs` and `statusline` plugins: rejected due to vertical
  grid cost (2 rows consumed out of available terminal height).
- Embedding a hardcoded Waybar-style widget directly in Core: rejected by
  ADR-0014; presentation widgets must remain extensible plugins outside Core.

## Related

- [Bar plugin documentation](README.md)
- [Schemas and contracts](schemas.md)
- [Evidence and links](evidence.md)
- [Statusline plugin design](../statusline/design.md)
- [ADR 0014: Workspace as Core Mechanism with Plugin-Only Presentation](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0014-workspace-core-presentation-plugins.md)
- [Sparse Workspaces and Unified Chrome Bar Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/sparse-workspace-unified-bar-candidate.md)
