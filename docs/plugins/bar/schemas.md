---
title: Bar plugin schemas and contracts
description: Manifest surface version ranges capabilities and configuration schemas for the bar plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 52
---

# Bar plugin schemas and contracts

## Manifest

| Field                 | Value                |
| --------------------- | -------------------- |
| `[plugin].id`         | `bitty-terminal.bar` |
| `[plugin].name`       | Unified Bar          |
| `[plugin].version`    | 0.0.1                |
| `[compat].bitty`      | `>=0.1,<1.0`         |
| `[compat].plugin-api` | `^1.0`               |

## Capabilities

`ui.chrome-band`, `terminal.semantic-read`. Deny by default; no allow-all
identifier exists. Requested capabilities are checked by the plugin host before
loading.

## Configuration schema

Configuration keys declared in `bitty.toml` under `[plugins."bitty-terminal.bar"]`:

| Key                          | Type            | Default                  | Description                                                |
| ---------------------------- | --------------- | ------------------------ | ---------------------------------------------------------- |
| `edge`                       | `string`        | `"bottom"`               | Band placement: `"top"` or `"bottom"`.                     |
| `modules_left`               | `array[string]` | `["workspaces", "mode"]` | Modules positioned at the start edge.                      |
| `modules_center`             | `array[string]` | `["window_title"]`       | Modules positioned centered in available space.            |
| `modules_right`              | `array[string]` | `["agents", "clock"]`    | Modules positioned at the end edge.                        |
| `workspaces.format`          | `string`        | `"{id}"`                 | Format string for sparse workspace pills.                  |
| `workspaces.show_empty`      | `boolean`       | `false`                  | Whether to render empty non-pinned workspace slots.        |
| `workspaces.show_add_button` | `boolean`       | `true`                   | Whether to display the `+` pill to allocate new workspace. |

## Observation events

Static triggers drive recomposition without polling:

| Event                    | Payload                                   | Purpose                                            |
| ------------------------ | ----------------------------------------- | -------------------------------------------------- |
| `workspace.snapshot`     | `{ active: u64, workspaces: array[u64] }` | Complete list of active sparse workspace IDs.      |
| `workspace.created`      | `{ id: u64 }`                             | Emitted when a new workspace is allocated.         |
| `workspace.destroyed`    | `{ id: u64 }`                             | Emitted when an empty dynamic workspace is pruned. |
| `focus.changed`          | `{ workspace_id: u64, panel_id: u64 }`    | Focus shifted between panels or workspaces.        |
| `terminal.cwd-changed`   | `{ panel_id: u64, cwd: string }`          | Working directory change in focused pane.          |
| `terminal.title-changed` | `{ panel_id: u64, title: string }`        | Title update from VT sequence or child process.    |
| `runtime.status`         | `{ headless_agents_active: u32 }`         | Count of background headless agent panels running. |

## Dispatched commands

Pointer interactions on the bar translate into standard Core workspace commands:

| Interaction           | Target Module  | Dispatched Command          | Effect                                     |
| --------------------- | -------------- | --------------------------- | ------------------------------------------ |
| Left-click            | Workspace `N`  | `workspace_focus:N`         | Switch focus to workspace `N` (or create). |
| Left-click            | Add button `+` | `workspace_new`             | Allocate and focus new sparse workspace.   |
| Middle-click          | Workspace `N`  | `workspace_close_request:N` | Request close of all sessions in `N`.      |
| Mouse wheel up / down | Workspaces     | `workspace_cycle:1` / `-1`  | Cycle through active sparse workspaces.    |

## Authority

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012)
and the candidate
[Sparse Workspaces and Unified Chrome Bar Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/sparse-workspace-unified-bar-candidate.md).
Manifest validity is checked by `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk).
