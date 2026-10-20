---
title: Plugin README user documentation
description: Canonical README structure and settings table contract for plugin repositories
category: specifications
audience: plugin-author
document_type: policy
status: normative
website_publish: true
sidebar_order: 31
---

# Plugin README user documentation

> Status: **normative** convention for every plugin repository. The executable
> contract lives in the `bitty-plugin-sdk` `bitty-plugin-docs-check` tool; this
> page records the convention and its rationale so plugin authors and
> reviewers share one definition. Owning task: the plugin README convention
> decision recorded in the plugin-ecosystem corpus.

## Purpose and scope

This policy fixes where a plugin's user documentation lives, which sections it
must contain, and how its configuration is documented. It applies to every
official and community plugin repository, including generated repositories
created from `bitty-plugin-template`.

In scope: the canonical file rule, the required top-level section order, the
Configuration settings table, and the checks that keep the table synchronized
with `bitty-plugin.toml [settings]`.

Out of scope: manifest schema and settings semantics, which the accepted
[Plugin Platform RFC](../specifications/plugin-platform-rfc.md) and the
[Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md) own, and
per-plugin design pages, which live under `docs/plugins/` in this corpus.

## Normative sources

- The Plugin Platform RFC fixes the manifest `[settings]` declaration shape and
  the per-key precedence: the manifest default, the valid shared host source,
  then the valid selected configuration file.
- The `bitty-plugin-sdk` `docs/user-documentation.md` page and the
  `bitty-plugin-docs-check` tool define the executable check and its diagnostic
  codes.
- This policy must not weaken either source. Where the checker and this page
  disagree, the checker is the executable truth and this page is corrected in
  the same change.

## Canonical file rule

Each plugin repository's root `README.md` is the single canonical user
document. A plugin repository must not add a `docs/` directory for
user-facing content: installation, commands, configuration, keybindings, and
troubleshooting stay in the README. Design and architecture material that needs
more space belongs in this documentation corpus under `docs/plugins/<plugin>/`,
not in a second user-doc tree inside the plugin repository.

The README is canonical for two concrete reasons:

1. GitHub renders the repository root README as the project landing page. A
   user who finds the plugin repository sees that file first.
2. The future plugin website ingests the same file. One canonical source keeps
   the rendered repository page and the website page from drifting into
   competing copies.

## Required section order

Six H2 sections lead the README, in this order:

1. `## Overview`
2. `## Install`
3. `## Commands`
4. `## Configuration`
5. `## Keybindings`
6. `## Troubleshooting`

Developer and architecture sections follow the six: `## Layout`,
`## Development`, `## Capabilities`, `## API contract`, `## Security`, and
similar. The H1 title and a short lede may precede `## Overview`.

Heading text matches exactly, including case. A required section that is
missing, out of order, or duplicated fails the check. Headings and tables
inside fenced code blocks are ignored, so examples do not satisfy the
structure.

## Commands

The `Commands` section lists the commands the plugin registers, normally as a
table with the qualified `<plugin-id>:<resource>` name and a one-line
description. The section is documentation, not a machine-checked surface; the
manifest `[lazy].commands` reservation remains the source of command identity.

## Configuration

The `Configuration` section carries one table row per key declared in
`bitty-plugin.toml` `[settings]`, in the manifest's declaration order:

| Setting    | Type   | Default   | Description                                                   |
| ---------- | ------ | --------- | ------------------------------------------------------------- |
| `greeting` | string | `"Hello"` | Greeting prefix printed by the `example.hello:hello` command. |

Rules:

- The `Setting` key set, the `Type` cell, and the row order are generated from
  `[settings]` and checked against it. A declared key missing from the table
  fails, and a table row naming a key the manifest does not declare fails.
- `Type` is the exact manifest spelling: `bool`, `int`, `string`, or
  `string-array`.
- `Default` is the shipped default rendered as a TOML-shaped literal:
  `true`, `32`, `"Hello"`, or `["a", "b"]`.
- `Description` is authored prose. The accepted `[settings]` shape carries only
  `type` and `default`, so the description cannot be generated.
- When the manifest declares no settings, write the exact sentence
  `This plugin declares no settings.` or keep an empty table header.

Generate the checked columns instead of writing them by hand:

```sh
bunx --bun --package github:bitty-terminal/bitty-plugin-sdk#<commit-sha> \
  bitty-plugin-docs-check --print-settings-table .
```

The plugin's aggregate gate runs the checker so the table cannot drift
silently:

```make
docs:
    bunx --bun --package github:bitty-terminal/bitty-plugin-sdk#{{ sdk_ref }} bitty-plugin-docs-check .

check: manifest lua lua-control docs
```

## Keybindings

The `Keybindings` section states the registered keybindings, or the exact fact
that none are registered. Suggested keybindings belong here too, marked as
suggestions: a plugin suggestion never overrides a user or workspace mapping.

## Troubleshooting

The `Troubleshooting` section covers the common failure modes: a missing
dependency, a manifest that fails validation, a settings key that stays at its
default, and a command that does not appear. Each entry names a symptom and a
concrete fix, not a general troubleshooting link.

## Template snippet

Replace the placeholders and keep the six H2 headings byte-identical. Generate
the Configuration table from `[settings]` and keep the Description cells.

```markdown
# <plugin name>

<one-line description>

## Overview

What the plugin does, in one or two paragraphs.

## Install

How to install and enable it, including the supported host range.

## Commands

| Command                 | Description            |
| ----------------------- | ---------------------- |
| `<plugin-id>:<command>` | What the command does. |

## Configuration

Settings use `plugins.<plugin-id>.*`. The table is generated from
`bitty-plugin.toml` `[settings]`.

| Setting | Type     | Default     | Description               |
| ------- | -------- | ----------- | ------------------------- |
| `<key>` | `<type>` | `<default>` | What the setting changes. |

## Keybindings

Registered keybindings, or that none are registered.

## Troubleshooting

Symptom and fix for the common failure modes.

## Development

Contributor-only instructions follow the six sections above.
```

## Security review

- The README is public prose. It must not contain credentials, tokens, private
  endpoints, or user-specific paths.
- Configuration examples show keys and shipped defaults only. They grant no
  authority; capability requests remain in the manifest and follow the
  deny-by-default capability model.
- The checker reads `bitty-plugin.toml` and `README.md` only. It has no
  network, process-spawn, or write authority, and it treats the manifest as
  untrusted input through the accepted validator.

## Verification

- Run `bitty-plugin-docs-check` from the SDK commit the plugin pins. A missing
  or extra settings key, a type mismatch, a wrong row order, or a
  missing/misordered required section exits non-zero.
- Wire the check into the plugin's aggregate `just check` so the same gate runs
  locally and in CI.
- Registry-time enforcement is not configured yet: official plugins predate
  this convention, so the registry validation treats README structure as a
  migration item rather than a hard gate until the corpus is updated.

## References

- [Plugin Platform RFC](../specifications/plugin-platform-rfc.md)
- [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md)
- [SDK index](README.md)
- [bitty-plugin-sdk user documentation](https://github.com/bitty-terminal/bitty-plugin-sdk/blob/main/docs/user-documentation.md)
