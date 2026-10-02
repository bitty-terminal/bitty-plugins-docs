---
title: Beacon plugin schemas and contracts
description: Candidate manifest config key-language scope filter and badge shapes for the Beacon targeting-policy plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 56
---

# Beacon plugin schemas and contracts

## Manifest

The manifest shape is the accepted one; every value below is candidate because
no package exists.

| Field                 | Value                                                    |
| --------------------- | -------------------------------------------------------- |
| `[plugin].id`         | `bitty-terminal.beacon` (candidate)                      |
| `[plugin].name`       | Beacon                                                   |
| `[plugin].version`    | Not created; `0.0.1` reserved by the repository standard |
| `[compat].bitty`      | `>=0.1,<1.0` (candidate)                                 |
| `[compat].plugin-api` | `^1.0` (candidate)                                       |

## Capabilities

Not defined. Beacon would request only capability-gated public surfaces:
target-provider registration, target observation, transient input capture
through the host API, and the declarative UI slot for hints or menus. The
capability identifiers, their dimensions, and their API version remain open
(OQ-056) and are owned by `W-29`/`W-120`. Deny by default applies; there is no
allow-all identifier, and an unknown identifier fails validation.

## Policy configuration

Candidate shape in `bitty.toml` under `[plugins."bitty-terminal.beacon"]`. Keys
and value shapes are **illustrative-only**; the accepted key surface belongs to
the plugin contract and `W-120`.

| Key          | Type            | Purpose (candidate)                                   |
| ------------ | --------------- | ----------------------------------------------------- |
| `bindings`   | `table`         | Chord to operator command, as a suggestion only.      |
| `operators`  | `table`         | Operator family to default scope and filter set.      |
| `scopes`     | `table`         | Named scope definitions and their default per family. |
| `filters`    | `table`         | Public-metadata predicates applied before labeling.   |
| `badges`     | `table`         | Badge token to theme token mapping.                   |
| `menu.order` | `array[string]` | Target-first menu grouping and ordering.              |

## Key-language table

Candidate mapping from an operator family to its default scope. Chords are
illustrative-only and are user-overridable suggestions.

| Operator family (candidate) | Illustrative chord | Default scope (candidate) | Notes                                   |
| --------------------------- | ------------------ | ------------------------- | --------------------------------------- |
| Focus                       | not fixed          | `CurrentWorkspace`        | Labels panels and workspaces.           |
| Fold                        | not fixed          | `CurrentPanel`            | Labels command blocks.                  |
| Move                        | not fixed          | `CurrentPanel`            | Numbered destination after the label.   |
| Target-first entry          | not fixed          | `Window`                  | Opens the declared-action context menu. |

## Scopes and filters

| Scope (candidate)     | Bounds targets to                                                |
| --------------------- | ---------------------------------------------------------------- |
| `CurrentControlGroup` | The focused control group, for example a form or toolbar region. |
| `CurrentPanel`        | The focused panel.                                               |
| `CurrentWorkspace`    | The active workspace.                                            |
| `Window`              | Everything visible in the active OS window.                      |
| `WorkspaceRail`       | The workspace indicator surface and its entries.                 |
| `SemanticDomain`      | A named domain, for example git or container objects.            |

Filters are predicates over public target metadata only. Scope kinds, their
spellings, and the filter expression grammar are candidate; the default scope
per operator and scope composition stay open (OQ-088).

## Theme badges

| Badge token (candidate) | Meaning                              | Theme source (candidate) |
| ----------------------- | ------------------------------------ | ------------------------ |
| `workspace`             | A workspace or workspace-rail entry. | Host theme token set.    |
| `agent`                 | A background or agent-owned target.  | Host theme token set.    |
| `block`                 | A semantic command block.            | Host theme token set.    |
| `link`                  | An addressable link or rich target.  | Host theme token set.    |

Badge metadata is public bounded data; the Core annotation layer and host theme
own rasterization. Token names and the theme-token namespace are candidate and
depend on `W-29`/`W-120`.

## Command and event surface

Not fixed. Commands would be registered through the accepted
`bitty.commands.register` surface and observation through the accepted closed v1
event-name set; any Beacon-specific command, event, or key-suggestion payload
requires `W-29`/`W-120`. No command id, event name, or payload shape is decided
here.

## Authority

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012),
with the manifest and capability grammar authority in
[Plugin Manifest and Capability Grammar Authority](../../../specifications/manifest-capability-authority.md).
The plugin must obey the accepted manifest, capability, grant, lifecycle, and
command-registry rules as an ordinary package with no special privilege.
Manifest validity is checked by `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk). The
owning repository, once created, holds the manifest file and its validation
evidence; no manifest exists today.

## Related

- [Beacon plugin documentation](README.md)
- [Beacon plugin design](design.md)
- [Beacon plugin evidence and links](evidence.md)
- [Beacon Targeting Framework (Candidate)](../../../specifications/beacon-targeting-framework-candidate.md)
- [Plugin API v1 Lua Surface RFC](../../../sdk/plugin-api-v1-lua-surface-rfc.md)
