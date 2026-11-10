---
title: File manager plugin schemas and contracts
description: Manifest surface version ranges capabilities commands and events for the file manager plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 44
---

# File manager plugin schemas and contracts

## Purpose and scope

Describe the read-only listing package at registry-pinned file-manager
`558bf5a2` (manifest
`sha256:b26f36ecbe7ce5b3fb0338c90c594f98a104de021a9b7eab4b6b6af8645fa010`).
Registry eligibility is separate from filesystem and panel usability.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and manifest validation. The
[settings reference](../../../sdk/reference/settings.md) owns the current
declared-key behavior; it accepts no table-shaped `entries` kind.

## Manifest

| Field                 | Value                         |
| --------------------- | ----------------------------- |
| `[plugin].id`         | `bitty-terminal.file-manager` |
| `[plugin].name`       | File Manager                  |
| `[plugin].version`    | 0.1.0                         |
| `[compat].bitty`      | `>=0.0.24`                    |
| `[compat].plugin-api` | `^1.0`                        |

## Capabilities

`terminal.semantic-read`, `ui.rich`, and one read-only filesystem scope. Deny by
default; unknown identifiers fail validation and there is no allow-all
identifier. `ui.rich` gates the `bottom` band block that renders the listing
through `bitty.ui.mount`/`update`; `terminal.semantic-read` gates the read-only
cwd/title observation. The filesystem declaration is
`access = "read"` with `paths = ["**"]`, which gates `bitty.fs.list`. No
`fs.write`, process, network, clipboard, or terminal-input authority is
requested.

## Lazy surface

| Trigger  | Values                                                            |
| -------- | ----------------------------------------------------------------- |
| Commands | `bitty-terminal.file-manager:open`, `:preview`, `:rename`         |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed` |

Static triggers let the host register the plugin without creating a VM; the
observation events refresh the cached snapshot-derived state. `:rename` is
policy-only composition (rename preview and selection with no host filesystem
mutation); any host mutation stays deferred pending the `fs.write:PATTERN`
grant and the panel-provider contract (`bitty-docs` `CTX-0181`, OQ-058),
consistent with the file-manager candidate in the Plugin Matrix.

## Configuration keys

The manifest declares three keys; values are configuration, never authority:

| Key       | Declared kind  | Default | Purpose                                                               |
| --------- | -------------- | ------- | --------------------------------------------------------------------- |
| `root`    | `string`       | `""`    | Scope root; empty falls back to the observed cwd.                     |
| `entries` | `string-array` | `[]`    | Candidate paths used instead of a host listing when non-empty.        |
| `width`   | `int`          | `0`     | Band width in cells; `0` derives it from the semantic snapshot width. |

`entries` is declared as a string array only. A table-shaped `entries` list (one
object per candidate with per-entry fields) remains an unaccepted, deferred
question in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90); this page
does not widen the schema to admit it, and the Lua reads a table only because
the host delivers a string array as an indexable value.

None of the three keys grants authority beyond `terminal.semantic-read` and the
`fs.read:**` scope: they select a root and a listing width, and cannot enumerate
anything the host would refuse.

## Policy bounds

Host-free Lua constants bound the composed listings: at most 128 entries per
listing, names at 128 code points, paths at 4096 bytes, selection at 64, and
the total listing payload at 8192 bytes (`8 KiB`).

## Security review

The package calls no panel API and requests no panel capability: it presents a
`bottom` band block, not a panel slot. Directory enumeration is host-mediated
through `bitty.fs.list` under the read-only `fs.read:**` scope, so the host's
scope, sensitive-path, secret, and budget gates apply to every prefix. `open`
shapes supplied paths, `preview` shapes path-derived data, and `rename` returns a
validated pair without mutating a file. Pure path tests do not prove real-path,
symlink, or device isolation of the host filesystem backend, and no live
filesystem campaign backs this page.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
